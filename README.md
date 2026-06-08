# MidasCore

A Spring Boot application that manages user balances and financial transactions, built as part of the JPMC MidasCore platform.

---

## Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Technologies Used](#technologies-used)
- [Getting Started](#getting-started)
- [API Reference](#api-reference)
- [Data Model](#data-model)
- [Known Issues & Notes](#known-issues--notes)

---

## Overview

MidasCore is a RESTful microservice that handles user account balances and transaction processing. It exposes HTTP endpoints to query balances, processes incoming transactions via a Kafka-based listener, and optionally applies incentives by calling an external service.

---

## Project Structure

```
com.jpmc.midascore/
├── MidasCoreApplication.java          # Spring Boot entry point
├── component/
│   └── DatabaseConduit.java           # Wrapper for persistence operations
├── controller/
│   └── BalanceController.java         # REST controller for balance queries
├── config/
│   └── RestTemplateConfig.java        # RestTemplate bean configuration
├── entity/
│   ├── UserRecord.java                # JPA entity for users
│   ├── TransactionRecord.java         # JPA entity for transaction history
│   └── Incentive.java                 # Model for incentive response payload
├── foundation/
│   ├── Transaction.java               # DTO for incoming transaction messages
│   └── Balance.java                   # DTO for balance API responses
└── repository/
    ├── UserRepository.java            # CRUD repository for UserRecord
    └── TransactionRecordRepository.java # JPA repository for TransactionRecord
```

---

## Technologies Used

| Technology | Purpose |
|---|---|
| Java | Primary language |
| Spring Boot | Application framework |
| Spring Data JPA | Database ORM and repository layer |
| Spring Web (RestTemplate) | HTTP client for external service calls |
| Jakarta Persistence (JPA) | Entity mapping and database interaction |
| Apache Kafka | (Expected) Message broker for transaction events |
| H2 / Relational DB | Persistent storage for users and transactions |

---

## Getting Started

### Prerequisites

- Java 17+
- Maven or Gradle
- A running Kafka broker (if transaction listener is active)
- A configured relational database (H2 for local dev, or configure `application.properties`)

### Running the Application

```bash
# Clone the repository
git clone <repository-url>
cd midascore

# Build the project
./mvnw clean install

# Run the application
./mvnw spring-boot:run
```

The application starts on `http://localhost:8080` by default.

---

## API Reference

### Get User Balance

Returns the current balance for a given user.

```
GET /balance?userId={userId}
```

**Query Parameters**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `userId` | `Long` | Yes | The ID of the user |

**Response**

```json
{
  "amount": 250.75
}
```

Returns `{ "amount": 0 }` if the user is not found.

**Example**

```bash
curl "http://localhost:8080/balance?userId=1"
```

---

## Data Model

### UserRecord

Represents a registered user with a balance.

| Field | Type | Description |
|---|---|---|
| `id` | `long` | Auto-generated primary key |
| `name` | `String` | User's name (non-null) |
| `balance` | `float` | Current account balance (non-null) |

### TransactionRecord

Persists a record of each processed transaction.

| Field | Type | Description |
|---|---|---|
| `id` | `Long` | Auto-generated primary key |
| `sender` | `UserRecord` | The user sending funds |
| `recipient` | `UserRecord` | The user receiving funds |
| `amount` | `float` | Transaction amount |

### Transaction (DTO)

Represents an incoming transaction message (e.g., from Kafka).

| Field | Type | Description |
|---|---|---|
| `senderId` | `long` | ID of the sender |
| `recipientId` | `long` | ID of the recipient |
| `amount` | `float` | Amount to transfer |

### Balance (DTO)

API response object for balance queries.

| Field | Type | Description |
|---|---|---|
| `amount` | `float` | The user's current balance |

### Incentive

Represents an incentive amount returned by an external incentive service.

| Field | Type | Description |
|---|---|---|
| `amount` | `float` | Incentive amount to apply to a transaction |

---

## Known Issues & Notes

- **Duplicate `RestTemplate` bean**: `RestTemplate` is declared as a `@Bean` in both `MidasCoreApplication.java` and `RestTemplateConfig.java`. One should be removed to avoid a bean conflict. Prefer keeping `RestTemplateConfig.java` for clean separation of concerns.

- **`TransactionRecord` defined in multiple files**: A `TransactionRecord` class appears in both `UserRecord.java` (without a package declaration and referencing a non-existent `User` class) and as its own separate file. The version in `UserRecord.java` should be removed, and `TransactionRecord.java` should be fixed to use `UserRecord` with the correct package import.

- **Missing Kafka listener**: The project includes `Transaction.java` as a DTO for incoming messages, but no `@KafkaListener` component is present in the uploaded source. A transaction processor component is likely needed to consume messages and apply balance updates.

- **`float` for monetary values**: Using `float` for financial amounts can introduce precision errors. Consider migrating to `BigDecimal` for production use.
