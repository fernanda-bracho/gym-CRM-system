# Gym CRM System — Spring Core Module

A Spring Core-based backend module designed to manage a Gym Customer Relationship Management (CRM) system, handling **Trainees**, **Trainers**, **Trainings**, and **Training Types** using in-memory storage, Spring Bean lifecycle hooks, and layered dependency injection.
(This is a homework task for epam java program)

---

## 📋 Table of Contents
- Overview
- Domain Model
- Architecture & Spring Requirements
- Business Rules
- Project Structure
- Configuration
- Getting Started
- [Testing

---

## 🎯 Overview

This module implements core CRM operations for a gym platform without relying on external databases or Spring Boot auto-configuration. Instead, it leverages **Spring Framework (Spring Core)** context configuration (Java-based or annotation-based), custom in-memory `Map` storage beans, and `BeanPostProcessor` initialization from external data files.

---

## 🗂️ Domain Model

The system revolves around five core domain entities:

* **`User`** *(Abstract / Base Entity)*
  * `id` / `userId` (`Long`)
  * `firstName` (`String`)
  * `lastName` (`String`)
  * `username` (`String`)
  * `password` (`String`)
  * `isActive` (`boolean`)
* **`Trainee`** *(extends `User`)*
  * `dateOfBirth` (`LocalDate`)
  * `address` (`String`)
  * `userId` (`Long`)
* **`Trainer`** *(extends `User`)*
  * `specialization` (`TrainingType`)
  * `userId` (`Long`)
* **`TrainingType`**
  * `trainingTypeName` (`String`)
* **`Training`**
  * `traineeId` (`Long`)
  * `trainerId` (`Long`)
  * `trainingName` (`String`)
  * `trainingType` (`TrainingType`)
  * `trainingDate` (`LocalDate`)
  * `trainingDuration` (`Number` / `Duration`)

---

## 🏗️ Architecture & Spring Requirements

### 1. Application Context
* Configured via **Java-based (`@Configuration`)** or **Annotation-based (`@ComponentScan`)** Spring Application Context.

### 2. Storage Layer (In-Memory Maps)
* Each entity (`Trainee`, `Trainer`, `Training`) is stored in and retrieved from a dedicated, separate in-memory `java.util.Map` Spring Bean (separate namespaces so each entity type can be listed independently).
* **Startup Initialization:** Storage beans are populated at application startup from a prepared data file using a Spring **`BeanPostProcessor`**.
* The path to the initial data file is injected using a **property placeholder (`@Value`)** backed by an external `.properties` file.

### 3. DAO Layer
* Dedicated DAO classes for **`Trainee`**, **`Trainer`**, and **`Training`** interacting directly with their respective in-memory storage beans.

### 4. Service Layer
* **`TraineeService`**: Supports **Create**, **Update**, **Delete**, and **Select** operations for Trainee profiles.
* **`TrainerService`**: Supports **Create**, **Update**, and **Select** operations for Trainer profiles.
* **`TrainingService`**: Supports **Create** and **Select** operations for Training sessions.

### 5. Facade Layer & Dependency Injection Rules
The project strictly enforces specific Spring dependency injection strategies across layers:
* **Storage Beans ➔ DAOs:** Setter-based injection (`@Autowired` on setters).
* **DAO Beans ➔ Services:** Field/setter autowiring (`@Autowired`).
* **Service Beans ➔ Facade:** **Constructor-based injection**.
* **All other dependencies:** **Setter-based injection**.

---

## ⚙️ Business Rules

### Credential Generation (Trainee & Trainer Creation)
When creating a new `Trainee` or `Trainer` profile:
1. **Username Calculation:**
   * Constructed by concatenating `firstName` and `lastName` separated by a dot (`.`), e.g., `John.Smith`.
   * **Collision Handling:** If a `Trainee` or `Trainer` with the same `firstName` and `lastName` pair already exists, a sequential serial number is appended as a suffix (e.g., `John.Smith1`, `John.Smith2`).
2. **Password Generation:**
   * Automatically generated as a random **10-character** alphanumeric string.

---

## 📁 Project Structure

```text
src
├── main
│   ├── java/com/gym/crm
│   │   ├── config/          # Spring @Configuration & PropertyPlaceholder config
│   │   ├── model/           # Domain entities (User, Trainee, Trainer, Training, TrainingType)
│   │   ├── storage/         # In-memory Map beans & BeanPostProcessor initializer
│   │   ├── dao/             # DAO implementations for Trainee, Trainer, Training
│   │   ├── service/         # Business logic & credential generation
│   │   └── facade/          # GymFacade orchestrating service calls (constructor-injected)
│   └── resources/
│       ├── application.properties  # External properties (data file path, etc.)
│       └── initial-data.json       # Prepared startup dataset (or CSV/properties format)
└── test
    └── java/com/gym/crm
        ├── dao/             # DAO unit tests
        ├── service/         # Service & credential rule unit tests
        ├── storage/         # BeanPostProcessor & storage seeding tests
        └── facade/          # Facade integration/unit tests
