# 🍃 Spring Core & Spring Boot Learning Repository

Welcome to my daily Java Spring Framework learning directory! This repository serves as a centralized documentation hub and hands-on code laboratory tracking core Spring concepts, architecture patterns, bean management, and framework internals.

---

## 📌 Syllabus & Topic Navigation

| ID | Topic | Key Concepts | Deep-Dive Link |
|:---:|:---|:---|:---:|
| **01** | **Inversion of Control & DI** | IoC Container, Dependency Injection Types | [`01-ioc-and-di.md`](./core-concepts/01-ioc-and-di.md) |
| **02** | **Annotations & Beans** | `@Component`, `@Bean`, `@Configuration`, Stereotypes | [`02-spring-annotations-and-beans.md`](./core-concepts/02-spring-annotations-and-beans.md) |
| **03** | **Bean Scopes** | Singleton, Prototype, Request, Session | [`03-bean-scopes.md`](./core-concepts/03-bean-scopes.md) |
| **04** | **Circular Dependency** | Startup cycles, `@Lazy`, Setter Injection | [`04-circular-dependency.md`](./core-concepts/04-circular-dependency.md) |
| **05** | **Bean Lifecycle** | `@PostConstruct`, `@PreDestroy`, Aware Hooks | [`05-bean-lifecycle.md`](./core-concepts/05-bean-lifecycle.md) |

---

## 🏗️ Core Architecture & Concept Breakdown

### 1. Spring IoC Container & Dependency Injection
The **Inversion of Control (IoC) Container** is the heart of Spring. Instead of manually creating objects using `new`, the container takes POJOs (Plain Old Java Objects) and configuration metadata to instantiate, configure, and wire your application components.
