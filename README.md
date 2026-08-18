# Spring Framework Concepts

A structured, theory-focused guide to the core concepts of the Spring ecosystem.

## About This Repository

This repository explains not only how Spring APIs are used, but also how their underlying concepts work and why they matter.

The articles move from the Spring container to web development, persistence, transactions, security, testing, and proxy-based infrastructure. Each section separates specifications, implementations, and Spring abstractions so that related technologies are not confused with one another.

Practical examples are intentionally small. Their purpose is to clarify the theory rather than build one complete application.

## Contents

### 01 — Spring Framework and Spring Boot

- [Spring Framework and Spring Boot: What Is the Difference?](01-what-is-spring-boot/spring-and-spring-boot.md)

### 02 — Spring MVC

- [Spring Framework, Spring MVC, and Spring Boot](02-spring-mvc-vs-spring-boot/spring-mvc.md)

### 03 — Spring Core

- [Spring Core Overview](03-spring-core/00-spring-core.md)
- [Inversion of Control and Dependency Injection](03-spring-core/01-ioc-and-dependency-injection.md)
- [Beans, Registration, and Autowiring](03-spring-core/02-beans-registration-and-autowiring.md)
- [Bean Lifecycle and Scopes](03-spring-core/03-bean-lifecycle-and-scopes.md)
- [BeanFactory and ApplicationContext](03-spring-core/04-beanfactory-and-applicationcontext.md)
- [Configuration, Annotations, and Environment](03-spring-core/05-configuration-annotations-and-environment.md)
- [Using Spring Core with Spring Boot](03-spring-core/06-spring-core-with-spring-boot.md)

### 04 — Spring Persistence

- [Persistence Overview](04-spring-persistence/00-persistence-overview.md)
- [JDBC, ORM, JPA, and Hibernate](04-spring-persistence/01-jdbc-orm-jpa-and-hibernate.md)
- [Entities and the Persistence Context](04-spring-persistence/02-entities-and-persistence-context.md)
- [Entity Lifecycle and Dirty Checking](04-spring-persistence/03-entity-lifecycle-and-dirty-checking.md)
- [Relationships, Fetching, and the N+1 Problem](04-spring-persistence/04-relationships-fetching-and-n-plus-one.md)
- [Spring Data JPA Repositories](04-spring-persistence/05-spring-data-jpa-repositories.md)
- [Query Methods, JPQL, and Native Queries](04-spring-persistence/06-query-methods-jpql-and-native-queries.md)
- [Using Spring Data JPA with Spring Boot](04-spring-persistence/07-spring-data-jpa-with-spring-boot.md)

### 05 — Spring Transactions

- [Transaction Management](05-spring-transactions/00-transaction-management.md)
- [Transactional Proxies, Propagation, and Isolation](05-spring-transactions/01-transactional-propagation-and-isolation.md)

### 06 — Spring Security

- [Spring Security Overview](06-spring-security/00-spring-security.md)
- [Filter Chain and Authentication](06-spring-security/01-filter-chain-and-authentication.md)
- [Authorization and Method Security](06-spring-security/02-authorization-and-method-security.md)
- [Sessions, JWT, OAuth 2.0, and Request Protection](06-spring-security/03-session-jwt-oauth2-and-protection.md)

### 07 — Testing Spring Applications

- [Testing Spring Applications](07-spring-testing/00-testing-spring-applications.md)
- [Test Slices and MockMvc](07-spring-testing/01-test-slices-and-mockmvc.md)
- [Integration Tests, Security Tests, and Testcontainers](07-spring-testing/02-integration-security-and-testcontainers.md)

### 08 — Spring AOP and Proxies

- [Spring AOP](08-spring-aop-and-proxies/00-spring-aop.md)
- [Proxy Boundaries and Common Limitations](08-spring-aop-and-proxies/01-proxy-limitations.md)

### 09 — Final Overview

- [Spring Ecosystem: Final Overview](09-final-overview/spring-ecosystem-overview.md)

## Reading Approach

Read the sections in order when learning the ecosystem for the first time. Later sections assume an understanding of beans, dependency injection, application contexts, and proxy-based interception.

When using the repository as a reference, start with the module responsible for the behavior you are investigating.

## Source Policy

The articles use primary sources wherever possible:

1. official Spring documentation
2. Jakarta specifications
3. official Hibernate, JUnit, Mockito, and Testcontainers documentation
4. additional educational material only as a secondary explanation

Each article contains references for its specific topic.

## Goal

The goal is to build a concise but technically accurate knowledge base that makes Spring applications easier to design, test, secure, and debug.

The repository also documents my continuous learning process and growing understanding of Spring-based backend development.
