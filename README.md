# Estrutura do Projeto
literAlura
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── example
│   │   │           └── literalura
│   │   │               ├── LiterAluraApplication.java
│   │   │               ├── config
│   │   │               │   └── WebConfig.java
│   │   │               ├── controller
│   │   │               │   └── BookController.java
│   │   │               ├── dto
│   │   │               │   ├── BookDto.java
│   │   │               │   └── BookForm.java
│   │   │               ├── model
│   │   │               │   ├── Author.java
│   │   │               │   ├── Book.java
│   │   │               │   └── Language.java
│   │   │               ├── repository
│   │   │               │   ├── AuthorRepository.java
│   │   │               │   ├── BookRepository.java
│   │   │               │   └── LanguageRepository.java
│   │   │               └── service
│   │   │                   ├── BookService.java
│   │   │                   └── GutendexService.java
│   │   └── resources
│   │       └── application.properties
└── pom.xml
# Dependências no pom.xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>
</dependencies>

# Configuração de Banco de Dados (application.properties)
spring.datasource.url=jdbc:postgresql://localhost:5432/literalura
spring.datasource.username=yourUsername
spring.datasource.password=yourPassword
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# Modelos (Author.java, Book.java, Language.java)
// Author.java
package com.example.literalura.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Author {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String birthYear;
    private String deathYear;
    // Getters and setters
}

// Book.java
package com.example.literalura.model;

import javax.persistence.*;
import java.util.Set;

@Entity
public class Book {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    private String downloads;

    @ManyToOne
    private Author author;

    @ManyToMany
    private Set<Language> languages;
    // Getters and setters
}

// Language.java
package com.example.literalura.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Language {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String code;
    // Getters and setters
}

# Repositórios (AuthorRepository.java, BookRepository.java, LanguageRepository.java)
// AuthorRepository.java
package com.example.literalura.repository;

import com.example.literalura.model.Author;
import org.springframework.data.jpa.repository.JpaRepository;

public interface AuthorRepository extends JpaRepository<Author, Long> {
}

// BookRepository.java
package com.example.literalura.repository;

import com.example.literalura.model.Book;
import org.springframework.data.jpa.repository.JpaRepository;

public interface BookRepository extends JpaRepository<Book, Long> {
}

// LanguageRepository.java
package com.example.literalura.repository;

import com.example.literalura.model.Language;
import org.springframework.data.jpa.repository.JpaRepository;

public interface LanguageRepository extends JpaRepository<Language, Long> {
}

# DTOs (BookDto.java, BookForm.java)
// BookDto.java
package com.example.literalura.dto;

import com.example.literalura.model.Book;

import java.util.List;
import java.util.stream.Collectors;

public class BookDto {
    private Long id;
    private String title;
    private String downloads;
    private String author;
    private List<String> languages;

    public BookDto(Book book) {
        this.id = book.getId();
        this.title = book.getTitle();
        this.downloads = book.getDownloads();
        this.author = book.getAuthor().getName();
        this.languages = book.getLanguages().stream()
                              .map(Language::getName)
                              .collect(Collectors.toList());
    }
    // Getters and setters
}

// BookForm.java
package com.example.literalura.dto;

import com.example.literalura.model.Book;
import com.example.literalura.repository.AuthorRepository;
import com.example.literalura.repository.LanguageRepository;

import javax.validation.constraints.NotEmpty;
import javax.validation.constraints.NotNull;
import java.util.Set;
import java.util.stream.Collectors;

public class BookForm {
    @NotNull @NotEmpty
    private String title;
    @NotNull @NotEmpty
    private String downloads;
    @NotNull @NotEmpty
    private String authorName;
    @NotNull @NotEmpty
    private Set<String> languageCodes;

    public Book convert(AuthorRepository authorRepository, LanguageRepository languageRepository) {
        var author = authorRepository.findByName(authorName);
        var languages = languageCodes.stream()
                                     .map(languageRepository::findByCode)
                                     .collect(Collectors.toSet());
        return new Book(title, downloads, author, languages);
    }
    // Getters and setters
}

# Controlador (BookController.java)
package com.example.literalura.controller;

import com.example.literalura.dto.BookDto;
import com.example.literalura.dto.BookForm;
import com.example.literalura.model.Book;
import com.example.literalura.repository.AuthorRepository;
import com.example.literalura.repository.BookRepository;
import com.example.literalura.repository.LanguageRepository;
import com.example.literalura.service.GutendexService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/books")
public class BookController {

    @Autowired
    private BookRepository bookRepository;

    @Autowired
    private AuthorRepository authorRepository;

    @Autowired
    private LanguageRepository languageRepository;

    @Autowired
    private GutendexService gutendexService;

    @GetMapping
    public List<BookDto> list() {
        var books = bookRepository.findAll();
        return books.stream().map(BookDto::new).collect(Collectors.toList());
    }

    @PostMapping
    @Transactional
    public ResponseEntity<BookDto> create(@Validated @RequestBody BookForm form) {
        var book = form.convert(authorRepository, languageRepository);
        bookRepository.save(book);
        return ResponseEntity.ok(new BookDto(book));
    }

    @GetMapping("/{id}")
    public ResponseEntity<BookDto> detail(@PathVariable Long id) {
        var book = bookRepository.findById(id);
        return book.map(value -> ResponseEntity.ok(new BookDto(value)))
                   .orElseGet(() -> ResponseEntity.notFound().build());
    }

    @DeleteMapping("/{id}")
    @Transactional
    public ResponseEntity<?> delete(@PathVariable Long id) {
        var book = bookRepository.findById(id);
        if (book.isPresent()) {
            bookRepository.delete(book.get());
            return ResponseEntity.ok().build();
        }
        return ResponseEntity.notFound().build();
    }

    @GetMapping("/title/{title}")
    @Transactional
    public ResponseEntity<BookDto> findByTitle(@PathVariable String title) {
        var book = gutendexService.findBookByTitle(title);
        if (book != null) {
            bookRepository.save(book);
            return ResponseEntity.ok(new BookDto(book));
        }
        return ResponseEntity.notFound().build();
    }
}

# Serviço de Consumo da API (GutendexService.java)
package com.example.literalura.service;

import com.example.literalura.model.Author;
import com.example.literalura.model.Book;
import com.example.literalura.model.Language;
import com.example.literalura.repository.AuthorRepository;
import com.example.literalura.repository.BookRepository;
import com.example.literalura.repository.LanguageRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

import java.util.HashSet;

@Service
public class GutendexService {

    @Autowired
    private AuthorRepository authorRepository;

    @Autowired
    private BookRepository bookRepository;

    @Autowired
    private LanguageRepository languageRepository;

    public Book findBookByTitle(String title) {
        String url = "https://gutendex.com/books?search=" + title;
        RestTemplate restTemplate = new RestTemplate();
        var response = restTemplate.getForObject(url, GutendexResponse.class);

        if (response != null && !response.getResults().isEmpty()) {
            var gutendexBook = response.getResults().get(0);
            var author = authorRepository.findByName(gutendexBook.getAuthor());
            if (author == null) {
                author = new Author();
                author.setName(gutendexBook.getAuthor());
                authorRepository.save(author);
            }

            var languages = new HashSet<Language>();
            for (String code : gutendexBook.getLanguages()) {
                var language = languageRepository.findByCode(code);
                if (language == null) {
                    language = new Language();
                    language.setCode(code);
                    languageRepository.save(language);
                }
                languages.add(language);
            }

            var book = new Book();
            book.setTitle(gutendexBook.getTitle());
            book.setDownloads(gutendexBook.getDownloads());
            book.setAuthor(author);
            book.setLanguages(languages);

            return book;
        }
        return null;
    }
}

# Modelo da Resposta da API (GutendexResponse.java)
package com.example.literalura.service;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

import java.util.List;

@JsonIgnoreProperties(ignoreUnknown = true)
public class GutendexResponse {
    private List<GutendexBook> results;

    // Getters and setters

    @JsonIgnoreProperties(ignoreUnknown = true)
    public static class GutendexBook {
        private String title;
        private String author;
        private String downloads;
        private List<String> languages;

        // Getters and setters
    }
}

# Iniciar a Aplicação (LiterAluraApplication.java)
package com.example.literalura;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class LiterAluraApplication {
    public static void main(String[] args) {
        SpringApplication.run(LiterAluraApplication.class, args);
    }
}
