# Alensi Flow

### Configurable Business Automation & Orchestration Platform

Alensi Flow is a configurable business automation and orchestration platform for designing, executing, and monitoring business processes across people, systems, APIs, events, and communication channels.

Instead of hard-coding every business process into application logic, Flow allows organizations to define reusable automations using triggers, conditions, actions, integrations, human tasks, schedules, and events.

Flow can orchestrate processes across Alensi services and external systems without requiring a new bespoke service for every business workflow.

---

## What This Project Demonstrates

- Configurable business automations
- Workflow and state-machine execution
- Event-driven orchestration
- Rule-based branching and conditions
- Human-in-the-loop processes
- Maker-checker and approval controls
- Scheduled automation
- API and service integrations
- External system connectors
- WhatsApp-driven automation
- Task assignment and escalation
- Workflow versioning
- Execution history and audit trails
- Idempotent workflow execution
- Retry and failure handling
- Observability and execution tracing

---

## The Core Idea

Flow separates **business process orchestration** from the domain systems that perform the actual business operations.

For example:

```text
                         Alensi Flow
                              |
             +----------------+----------------+
             |                |                |
          Triggers          Rules            Actions
             |                |                |
        Webhooks          Conditions        REST APIs
        Schedules         Branching         gRPC
        Events            Expressions       Events
        WhatsApp                           WhatsApp
        Manual                              Email
```

A workflow can therefore coordinate multiple systems without owning their business logic.

For example:

Customer Request
       |
       v
     Flow
       |
       +----> Identity
       |
       +----> LMS / CRM
       |
       +----> Payment Service
       |
       +----> Notification Channel
       |
       v
    Response

Example: WhatsApp Liquidity Inquiry

A financial institution could configure Flow to respond to customers through WhatsApp.

A customer sends:

Balance

Flow can execute:

WhatsApp Message
       |
       v
Resolve Customer
       |
       v
Check Identity
       |
       v
Call Financial System
       |
       v
Retrieve Liquidity
       |
       v
Transform Response
       |
       v
Send WhatsApp Message

The important part is that this does not require a dedicated backend implementation for every customer interaction.

The same automation engine can support:

Check liquidity balance
Check loan status
Request account statement
Check repayment date
Request payment instructions
Notify customers of account events
Escalate conversations to an agent

The business process is configuration; the underlying systems remain responsible for their domain operations.

Example: Payment Automation

Flow can orchestrate payment-related processes without becoming the payment system itself.

Payment Created
      |
      v
Risk Check
      |
      v
Amount > Approval Threshold?
      /       \
   +--+--+
   |     |
  Yes    No
   |     |
   v     v
Maker    Process
Review   Payment
   |
   v
Checker Approval
   |
   v
Payment Service

Alensi Pay remains responsible for payment execution.

Alensi Flow coordinates the process around it.

Example: Insurance Claim

Flow can orchestrate a configurable claims process:

Claim Submitted
       |
       v
Coverage Validation
       |
       v
AI Assessment
       |
       v
Risk / Anomaly Check
       |
       +------ High Risk ------> Human Review
       |
       v
Approval
       |
       v
Payment
       |
       v
Reconciliation
       |
       v
Claim Completed

The insurance domain remains in Alensi Insure.

Flow owns the process orchestration.

## Core Concepts

### Triggers

A workflow can begin from different types of events.

Supported trigger types can include:

HTTP webhook
REST API request
WhatsApp message
Scheduled execution
Cron expression
Domain event
Kafka event
RabbitMQ message
Manual execution
External system event

Example:

trigger:
  type: WHATSAPP_MESSAGE
  condition:
    message: "balance"

### Conditions & Branching

Business processes frequently need decisions.

Flow supports configurable conditions such as:

balance.available > 100000

or:

payment.amount >= 500000

A workflow can then branch:

                Payment
                   |
            Amount >= 500K?
              /          \
            Yes           No
             |             |
        Approval        Automatic
          Flow            Flow

### Actions

Actions represent operations that Flow can execute.

Examples include:

HTTP requests
gRPC calls
Publish an event
Send WhatsApp message
Send email
Create a task
Assign a task
Wait
Retry
Call another workflow
Update workflow state
Execute a configured connector

### Connectors

Flow uses connectors to interact with external systems.

A connector can encapsulate:

Authentication
Base URL
API operations
Request mapping
Response mapping
Retry policy
Timeout configuration
Error handling

Example:

connector:
  name: LendingSystem

  authentication:
    type: OAUTH2

  operations:
    - name: getLiquidity
      method: GET
      path: /customers/{customerId}/liquidity

    - name: getLoanStatus
      method: GET
      path: /customers/{customerId}/loans/status

This allows different clients to connect their own systems to Flow without changing the workflow engine itself.

### Human-in-the-Loop

Not every decision should be automated.

Flow supports human tasks when a process requires review, approval, or intervention.

Automated Processing
        |
        v
   Risk Assessment
        |
        v
   Requires Review?
      /       \
    No         Yes
    |           |
    v           v
Continue     Human Task
                |
                v
          Approve / Reject
                |
                v
             Continue

Tasks can be assigned based on:

Role
Permission
Organization
Region
Branch
User
Business rules

Authorization is delegated to Alensi Identity.

### Workflow Configuration

Workflows should be defined as configuration rather than hard-coded application logic.

Example:

workflow:
  name: payment-approval
  version: 3

  trigger:
    type: PAYMENT_CREATED

  steps:

    - id: check-amount
      type: CONDITION
      expression: payment.amount >= 500000

    - id: manager-approval
      type: HUMAN_TASK
      requiredPermission: PAYMENT_APPROVE

    - id: execute-payment
      type: API_CALL
      connector: payment-service
      operation: executePayment

    - id: notify-customer
      type: WHATSAPP_MESSAGE
      template: payment-success

The same engine can execute completely different workflows without changing its core implementation.

### Workflow Versioning

Business processes change.

Flow treats workflow definitions as versioned artifacts.

payment-approval

v1
v2
v3

Existing executions continue using the version under which they started, while new executions can use the latest active version.

This provides predictable behavior and auditability.

### Reliability

Long-running business processes introduce failure scenarios.

Flow is designed around:

Idempotent execution
Retry policies
Exponential backoff
Timeouts
Dead-letter handling
Execution checkpoints
Duplicate-event protection
Failure recovery
Compensation actions
Execution state persistence

A failed external API call should not necessarily mean the entire workflow is lost.

### Execution Model

A workflow execution maintains its own state and context.

Workflow Definition
        |
        v
Workflow Execution
        |
        +-- Context
        +-- Current Step
        +-- Variables
        +-- Execution History
        +-- Retry State
        +-- Pending Tasks
        +-- Errors

Example execution:

{
  "executionId": "WF-92831",
  "workflow": "payment-approval",
  "version": 3,
  "status": "WAITING_FOR_APPROVAL",
  "currentStep": "manager-approval"
}

### Event-Driven Automation

Flow can react to events emitted by other systems.

PaymentFailed
      |
      v
     Flow
      |
      +--> Wait 5 minutes
      |
      +--> Retry
      |
      +--> Still Failed?
              |
             Yes
              |
              +--> Create Support Task
              +--> Notify Customer

This allows Flow to coordinate asynchronous business processes across distributed systems.

## Alensi Ecosystem

Flow acts as an orchestration layer across the Alensi platform.

                         Alensi Gateway
                               |
          +--------------------+--------------------+
          |                    |                    |
      Identity                Pay                Insure
          |                    |                    |
          +--------------------+--------------------+
                               |
                          Alensi Flow
                               |
                    Business Processes
                               |
                     +---------+---------+
                     |                   |
                   Recon              External
                                       Systems

### Alensi Identity

Provides:

Authentication
Authorization
Roles
Permissions
Organizational scope

### Alensi Pay

Provides:

Payment execution
Provider orchestration
Payment status
Refunds
Provider integrations

### Alensi Recon

Provides:

Transaction ingestion
Matching
Reconciliation
Exception management

### Alensi Insure

Provides:

Policies
Claims
Coverage
Insurance intelligence

### Alensi Flow

Coordinates processes across these capabilities.

It does not replace them.

## Example Business Automation

A client could configure:

WHEN
    PaymentFailed

IF
    retryCount < 3

THEN
    Wait 5 minutes

THEN
    Retry Payment

IF
    retryCount >= 3

THEN
    Create Support Task

THEN
    Notify Customer

No new bespoke microservice is required for this particular process.

The process is represented as configuration and executed by the Flow engine.

## Technology Stack

Java
Spring Boot
Spring WebFlux
PostgreSQL
Redis
RabbitMQ / Kafka
gRPC
WhatsApp Business API
OpenTelemetry
Docker
Kubernetes
GitHub Actions

## Engineering Focus

Alensi Flow focuses on:

Configurable business processes
Distributed workflow execution
Event-driven architecture
Long-running transactions
Human-in-the-loop automation
Integration orchestration
Reliable asynchronous processing
Idempotency
Failure recovery
Workflow versioning
Auditability
Multi-tenant configuration
Observability

## Project Status

This project is being developed as a production-oriented reference implementation exploring how configurable business automation can be designed for modern distributed systems.

The goal is not to implement a single fixed business workflow.

The goal is to build a reusable automation engine capable of adapting to different organizations, processes, integrations, and communication channels.

## Documentation

Architecture and engineering documentation will cover:

System architecture
Workflow execution model
Workflow definition model
State machines
Rule and condition evaluation
Connector architecture
Event-driven processing
Reliability and failure handling
Workflow versioning
Multi-tenancy
Authorization
Audit trails
API specification
Sequence diagrams
Architecture Decision Records

## Getting Started

Clone the repository.
Configure the required environment variables.
Start the infrastructure dependencies.
Run the Spring Boot application.
Define or import a workflow.
Trigger a workflow.
Inspect its execution state and history.
