# Spring Security API

## Introdução

### Objetivos do projeto
Para a criação de sistemas, é imprescindível a funcionalidade de autenticação para que suas páginas estejam 
devidamente protegidas. Portanto, o objetivo deste projeto é fornecer um guia para que a proteção e segurança 
de um sistema seja devidamente implementada.

### JWT - JSON Web Token
JWTs tornaram-se um método popular de proteger aplicações web modernas. Basicamente, os JWTs nos permite 
transmitir informações de forma segura entre partes através de um objeto JSON compacto, independente e 
assinado digitalmente.

## Passo a passo

### Passo 0: Configurar o banco de dados em `application.yml`

Neste marco zero, devemos configurar o banco de dados desejado por meio do arquivo `application.properties`, o
qual renomeei para `application.yml`. Abaixo, segue um exemplo de arquivo de configuração (em yml):
````
    spring:
        datasource:
            url: jdbc:mysql://localhost:3306/springsecurityapi
            username: root
            password: root
            driver-class-name: com.mysql.cj.jdbc.Driver
        jpa:
            hibernate:
                ddl-auto: create-drop
            show-sql: true
            properties:
                hibernate:
                    format_sql: true
                database: mysql
                database-platform: org.hibernate.dialect.MySQL8Dialect
````

### Passo 1: Criar entidades no pacote de models
Entidades nada mais são que tabelas em um banco de dados relacional (SQL). Para que o Spring boot reconheça a 
classe como entidade, podemos empregar a anotação `@Entity`, seguida da anotação `@Table` com o nome da tabela
como parâmetro.
    
Além disso, devemos especificar quais serão as colunas do banco de dados e seus respectivos nomes, empregando-se
a anotação `@Column`.

Por fim, devemos definir os Getters e Setters para todos os atributos da classe de entidade. 

Segue um exemplo de classe de entidade: 

````java
package com.henriquecostadev.springsecurityapi.model;

import jakarta.persistence.*;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.Collection;
import java.util.List;

@Entity
@Table(name = "user")
public class User implements UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Integer id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @Column(name = "username")
    private String username;

    @Column(name = "password")
    private String password;

    @Enumerated(value = EnumType.STRING)
    private Role role;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Role getRole() {
        return role;
    }

    public void setRole(Role role) {
        this.role = role;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority(role.name()));
    }

}
````

Neste exemplo, ainda, implementei a interface UserDetails, nativa do Spring Security, para definir os papéis
de cada usuário na aplicação (User Roles).

### Passo 2: Criar a camada de repositório 

Repositórios Spring são bem similares ao padrão de classes DAO (Data Access Object), onde cada classe deste 
tipo é responsável por fornecer operações CRUD nas tabelas do banco de dados. Seu objetivo principal é abstrair 
o acesso a dados de forma genérica a partir do seu modelo.

No exemplo abaixo, novamente, utilizei uma classe preexistente do JPA, denominada JpaRepository, onde implementei o
método `findBy`:

````java
package com.henriquecostadev.springsecurityapi.repository;

import com.henriquecostadev.springsecurityapi.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface UserRepository extends JpaRepository<User, Integer> {
    Optional<User> findByUsername(String username);
}
````

### Passo 3 - Criar as classes de serviço

A anotação `@Service` é utilizada em classes que fornecem funcionalidades pertencentes a regras de
negócio. O principal objetivo dessa classe é sobrescrever os métodos definidos em interfaces de 
repositório, para que assim seja descrita a lógica do negócio. 

Abaixo, segue um exemplo de como é geralmente empregada a classe de serviço:
````java
package com.henriquecostadev.springsecurityapi.service;

import com.henriquecostadev.springsecurityapi.repository.UserRepository;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service
public class UserDetailsServiceImpl implements UserDetailsService {


    private final UserRepository userRepository;

    public UserDetailsServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return userRepository.findByUsername(username).orElseThrow(()-> new UsernameNotFoundException("User not found."));
    }
}
````

--- Em breve ---
### Passo 4: Criar a classe de serviço JWT

#### 4.1. Extrair o payload (Claim) do token

#### 4.2. Extrair a informação desejada pelo payload

#### 4.3. Validar o token

#### 4.4. Verificar se o token está expirado


### Passo 5: Criar o JWT Filter

### Passo 6: Configurar o Spring Security

### Passo 7: Criar o Authentication Service

### Passo 8: Criar o Authentication Controller