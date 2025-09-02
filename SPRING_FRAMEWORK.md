# Introduction to Spring Framework

Congratulations! You have completed this module. This document summarizes the key concepts you have learned about the Spring Framework.

---

## Overview of Spring

- Spring is a comprehensive ecosystem supporting web apps, microservices, and data processing tools.
- Widely used in industries such as e-commerce, banking, healthcare, and social media for its scalability, security, and transaction management.
- Spring Boot accelerates web app and microservice development.
- Simplifies Java development by:
  - Automating dependency management
  - Offering built-in features
  - Promoting modularity
- Simplifies Java deployment by:
  - Automating configurations
  - Managing dependencies
  - Handling errors efficiently

---

## Core Components of Spring

- **Spring Core**
- **Spring MVC**
- **Spring AOP**

### Other Key Spring Tools

- Spring Data
- Spring REST Docs
- Spring Boot
- Spring Cloud

---

## Spring Application Structure

- **Beans**: Objects managed by Spring’s IoC container providing functionality in the application.
- **Application Context**: Central manager for beans and dependencies throughout the lifecycle.
- **Configuration**: Defines how components are set up using:
  - XML
  - Java annotations
  - Java classes
- **Dependencies**: Objects required by other objects to function properly.
- **Inversion of Control (IoC)**: Shifts dependency management to Spring.
- **Dependency Injection (DI)**: Spring injects dependencies at runtime instead of manual creation.
- **Autowiring**: Automatically injects dependencies based on type or name.
- **Aspect-Oriented Programming (AOP)**: Separates cross-cutting concerns such as logging, security, and transaction management.

---

## IntelliJ IDEA Overview

- Common IDE for Java and Spring development.
- Community edition is free and open-source.
- Features:
  - User-friendly interface
  - Code completion
  - Debugging tools
  - Version control integration
  - Refactoring tools
  - Built-in terminal
  - Plug-in support
  - Customization and accessibility
  - Privacy and security
- Key development features:
  - Productive development
  - Fast coding
  - Decompiler
  - Search tools
  - AI-powered tools
  - Debugging and performance
- Maintenance features:
  - Version control and collaboration
  - Database and web support
  - Framework support
  - Testing
  - Deployment tools

---

## Spring Annotations

- **Purpose**: Provide metadata and automate tasks like bean creation, request handling, and dependency injection.
- **Common Annotations**:
  - `@Component`: Declares a Spring-managed bean
  - `@Controller`: Defines a web controller in MVC
  - `@RestController`: Combines `@Controller` and `@ResponseBody`
  - `@Bean`: Registers a method’s return as a Spring bean
  - `@Configuration`: Defines a configuration class
  - `@RequestMapping`: Maps HTTP requests
  - `@GetMapping`: Maps HTTP GET requests
  - `@Autowired`: Injects dependencies automatically
  - `@Qualifier`: Specifies which bean to inject
  - `@Value`: Injects values from properties or environment
  - `@Scope`: Sets bean lifecycle and scope
  - `@PathVariable`: Extracts values from URI path
  - `@RequestParam`: Retrieves query parameters
  - `@ResponseBody`: Sends method return as response body

- **Annotation Levels**:
  - Class-level
  - Method-level
  - Field-level
  - Constructor and parameter-level

---

## Maven Overview

- **Purpose**: Project management and build automation tool for Java.
- Automates:
  - Dependency management
  - Compiling code
  - Running tests
  - Packaging applications
- **Key Aspects**:
  - Project Object Model (POM) file
  - Dependencies
  - Repositories
  - Build lifecycle
  - Plugins
- **pom.xml** contains:
  - Configuration details
  - Project build process
  - Dependencies
  - Build processes

- **IntelliJ & Maven Integration**:
  - Creating a Maven project
  - Adding dependencies
  - Running Maven goals
  - Managing plugins

---

## Steps to Define a Spring Project

1. **Set up the environment**  
   - Install JDK and Maven  
   - Verify installations

2. **Create a Maven project**  
   - Using command line or IDE (IntelliJ/Eclipse)

3. **Follow structured project layout**  
   - `src` directory for source code and tests  
   - `pom.xml` for dependencies

4. **Add required dependencies in `pom.xml`**  
   - Enable Spring core and web functionalities

5. **Create a basic Spring application**  
   - Configuration class  
   - Simple bean  
   - Main class to initialize Spring

6. **Compile and execute the application**  
   - Using Maven or IDE

---
