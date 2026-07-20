# Object-Oriented Design for the Integrated Water Bill Payment and Notification System

## Overview

The system is designed using **Object-Oriented Programming (OOP)** principles and follows a **Microservices Architecture**. Each microservice contains its own domain objects, business logic, and data access layer.

To improve maintainability, scalability, and extensibility, the system incorporates several software **Design Patterns**.

---

# Software Architecture

- **Architecture Style:** Microservices
- **Programming Paradigm:** Object-Oriented Programming (OOP)
- **Communication:** REST APIs and Message Queue (RabbitMQ/Kafka)
- **Database:** PostgreSQL
- **Caching:** Redis
- **Deployment:** Docker + Kubernetes

---

# Design Patterns Used

| Design Pattern | Purpose | Applied In |
|---------------|---------|------------|
| Strategy | Support multiple payment methods | Payment Service |
| Observer | Automatically send notifications | Notification Service |
| Repository | Separate business logic from database operations | All Services |
| Adapter | Integrate with existing water company billing system | Billing Service |
| Factory | Create payment gateway objects | Payment Service |
| Facade | Simplify payment workflow | Payment Service |
| Singleton | Configuration manager and logging | Entire System |

---

# Domain Model

```text
                    User
          ┌──────────┴──────────┐
          │                     │
     Customer            Administrator
          │
          │
     WaterAccount
          │
     ┌────┴─────┐
     │          │
    Bill     Payment
     │          │
     │          ▼
     │      Receipt
     │
     ▼
Notification
```

---

# Class Descriptions

## User (Base Class)

### Attributes

- userId
- username
- password
- email
- phone
- role
- status

### Methods

- login()
- logout()
- updateProfile()
- changePassword()

---

## Customer

**Inherits:** User

### Attributes

- customerId
- accountNumber
- address

### Methods

- viewBills()
- makePayment()
- viewPaymentHistory()
- downloadReceipt()

---

## Administrator

**Inherits:** User

### Methods

- manageUsers()
- generateReports()
- manageBills()
- monitorTransactions()

---

## WaterAccount

Represents the customer's water service account.

### Attributes

- accountNumber
- meterNumber
- balance
- status

### Methods

- retrieveBills()
- calculateBalance()
- updateBalance()

---

## Bill

Represents a monthly water bill.

### Attributes

- billId
- amount
- month
- dueDate
- status
- waterUsage

### Methods

- generateBill()
- calculateBill()
- markPaid()

---

## Payment

Represents a customer payment.

### Attributes

- paymentId
- amount
- paymentMethod
- paymentDate
- transactionReference
- status

### Methods

- processPayment()
- verifyPayment()
- generateReceipt()

---

## Receipt

Represents proof of payment.

### Attributes

- receiptId
- paymentReference
- amount
- receiptDate

### Methods

- downloadPDF()
- sendReceipt()

---

## Notification

Represents customer notifications.

### Attributes

- notificationId
- recipient
- message
- notificationType
- dateSent

### Methods

- sendSMS()
- sendEmail()
- sendPushNotification()

---

## Report

Represents system reports.

### Attributes

- reportId
- reportType
- generatedDate

### Methods

- generate()
- exportPDF()
- exportExcel()

---

# Supporting Classes

These classes support the business logic but are not domain entities.

---

## PaymentGateway (Strategy Pattern)

Provides a common interface for payment providers.

### Methods

- pay()
- verify()

### Implementations

```text
                PaymentGateway
                       ▲
      ┌────────────────┼────────────────┐
      │                │                │
 AirtelMoney      MpambaPayment     BankPayment
```

---

## PaymentFactory (Factory Pattern)

Creates payment gateway objects.

### Method

- createGateway()

Example:

```java
PaymentGateway gateway =
PaymentFactory.createGateway("Mpamba");
```

---

## BillingAdapter (Adapter Pattern)

Connects the new application to the existing water company billing system.

### Methods

- getCustomer()
- retrieveBills()
- updatePayment()

```text
Our System
      │
      ▼
 Billing Adapter
      │
      ▼
Existing Water Company Billing System
```

---

## NotificationManager (Observer Pattern)

Automatically notifies subscribed services when important events occur.

### Observers

- SMSNotification
- EmailNotification
- PushNotification

```text
Bill Generated
      │
      ▼
Notification Manager
      │
 ┌────┼─────┐
 ▼    ▼     ▼
SMS Email Push
```

---

## Repository Classes (Repository Pattern)

Responsible for database operations.

Repositories include:

- UserRepository
- CustomerRepository
- WaterAccountRepository
- BillRepository
- PaymentRepository
- ReceiptRepository
- NotificationRepository
- ReportRepository

Example:

```java
Bill bill = billRepository.findById(id);
```

---

## PaymentFacade (Facade Pattern)

Provides a simplified interface for the complete payment process.

Instead of calling:

- Billing Service
- Payment Service
- Receipt Service
- Notification Service

The client simply calls:

```java
paymentFacade.payBill();
```

---

## ConfigurationManager (Singleton Pattern)

Only one instance exists throughout the application.

Responsibilities:

- Database configuration
- API keys
- System settings
- Environment variables

---

# Class Relationships

```text
                          User
                  ┌────────┴────────┐
                  │                 │
                  ▼                 ▼
             Customer       Administrator
                  │
                  ▼
            WaterAccount
                  │
          ┌───────┴────────┐
          ▼                ▼
        Bill          PaymentHistory
          │
          ▼
      Payment
          │
          ▼
      Receipt
          │
          ▼
 NotificationManager
    ┌─────┼────────┐
    ▼     ▼        ▼
 SMS   Email   PushNotification

Payment
    │
    ▼
PaymentFactory
    │
    ▼
PaymentGateway (Strategy)
    ├──────── AirtelMoney
    ├──────── Mpamba
    ├──────── Visa
    └──────── BankTransfer

BillingService
      │
      ▼
BillingAdapter
      │
      ▼
Existing Water Company Billing System
```

---

# Mapping Objects to Microservices

## Authentication Service

Objects:

- User
- Customer
- Administrator

Patterns:

- Repository
- Singleton

---

## Billing Service

Objects:

- WaterAccount
- Bill

Patterns:

- Repository
- Adapter

---

## Payment Service

Objects:

- Payment
- Receipt
- PaymentGateway
- PaymentFactory
- PaymentFacade

Patterns:

- Strategy
- Factory
- Facade

---

## Notification Service

Objects:

- Notification
- NotificationManager

Pattern:

- Observer

---

## Reporting Service

Objects:

- Report

Patterns:

- Repository
- Builder (optional)

---

# Object Interaction Flow

```text
Customer
    │
    ▼
Login
    │
    ▼
View Bill
    │
    ▼
PaymentFacade
    │
    ▼
PaymentFactory
    │
    ▼
PaymentGateway
    │
    ▼
Payment Successful
    │
    ▼
BillingAdapter
    │
    ▼
Existing Water Company System Updated
    │
    ▼
NotificationManager
    │
 ┌──┴────────────┐
 ▼               ▼
SMS          Email
    │
    ▼
Receipt Generated
```

---

# Benefits of the Design

- Modular and easy to maintain.
- Supports future payment providers without modifying existing code.
- Easy integration with the existing water company billing system.
- Scales horizontally using microservices.
- Promotes code reuse through object-oriented principles.
- Improves testability using repositories and interfaces.
- Automatically notifies customers using the Observer pattern.
- Simplifies complex payment workflows using the Facade pattern.
- Supports future expansion to millions of users.