# Micro E-Commerce Service

Micro E-Commerce Service is a microservice application for managing an online vegetable store. This project is built using Go with a microservice architecture pattern, implementing event-driven communication via RabbitMQ.

## Table of Contents

-   [Architecture Overview](#architecture-overview)
-   [Microservices](#microservices)
-   [Prerequisites](#prerequisites)
-   [Tech Stack](#tech-stack)
-   [Installation](#installation)
-   [Configuration](#configuration)
-   [Running the Application](#running-the-application)
-   [API Documentation](#api-documentation)
-   [Database Migrations](#database-migrations)
-   [Testing](#testing)
-   [Project Structure](#project-structure)
-   [Contributing](#contributing)
-   [License](#license)

## Architecture Overview

This project follows a microservice architecture with the following services:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  User Service   │     │ Product Service │     │  Order Service  │
│    :8090        │────▶│     :8082       │────▶│     :8083       │
└─────────────────┘     └─────────────────┘     └─────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                          RabbitMQ                               │
└─────────────────────────────────────────────────────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│ Payment Service │     │Notification Svc │     │  Elasticsearch  │
│    :8084        │     │     :8081       │     │     :9200       │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Microservices

### 1. User Service (Port: 8090)

-   User authentication and authorization
-   User profile management
-   JWT token generation and validation
-   Database: PostgreSQL (Port: 5432)

### 2. Product Service (Port: 8082)

-   Product catalog management
-   Stock management
-   Product search via Elasticsearch
-   Database: PostgreSQL (Port: 5433)

### 3. Order Service (Port: 8083)

-   Order creation and management
-   Order status tracking
-   Order history
-   Database: PostgreSQL (Port: 5434)

### 4. Payment Service (Port: 8084)

-   Payment processing
-   Payment method management
-   Payment status tracking
-   Database: PostgreSQL (Port: 5435)

### 5. Notification Service (Port: 8081)

-   Real-time notifications via WebSocket
-   Email notifications
-   Push notifications
-   Database: PostgreSQL (Port: 5436)

## Prerequisites

Before starting, make sure your system has:

-   **Go 1.19 or higher** - [Download Go](https://golang.org/dl/)
-   **PostgreSQL** - Database for each service
-   **RabbitMQ** - Message broker for inter-service communication
-   **Redis** - Caching and session management
-   **Elasticsearch 7.17.10** - Product search and indexing
-   **Docker & Docker Compose** (optional) - For containerized deployment
-   **Make** - Build automation tool

## Tech Stack

-   **Language**: Go (Golang)
-   **Framework**: Fiber (HTTP framework)
-   **Database**: PostgreSQL
-   **Cache**: Redis
-   **Message Broker**: RabbitMQ
-   **Search Engine**: Elasticsearch
-   **Migration Tool**: golang-migrate
-   **Containerization**: Docker & Docker Compose

## Installation

### Using Docker Compose (Recommended)

1. Clone the repository:

```bash
git clone <repository-url>
cd micro-ecommerce-api
```

2. Start all services:

```bash
docker-compose up -d
```

This will start all microservices along with their dependencies (PostgreSQL, Redis, RabbitMQ, Elasticsearch).

### Manual Installation

1. Clone the repository:

```bash
git clone <repository-url>
cd micro-ecommerce-api
```

2. Install dependencies for all services:

```bash
make mod-all
```

Or individually:

```bash
make mod-tidy
make mod-download
```

## Configuration

### Environment Variables

Each service requires its own `.env` file. Copy the example file and adjust the configuration:

```bash
# For each service
cd user-service && cp .env.example .env
cd ../product-service && cp .env.example .env
cd ../order-service && cp .env.example .env
cd ../payment-service && cp .env.example .env
cd ../notification-service && cp .env.example .env
```

### Common Environment Variables

Each service typically requires:

```env
# Database Configuration
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=lokal
DB_NAME=service_name

# Redis Configuration
REDIS_HOST=localhost
REDIS_PORT=6379

# RabbitMQ Configuration
RABBITMQ_HOST=localhost
RABBITMQ_PORT=5672
RABBITMQ_USER=guest
RABBITMQ_PASS=guest

# Elasticsearch Configuration (for product and order services)
ELASTICSEARCH_URL=http://localhost:9200

# JWT Configuration (for user service)
JWT_SECRET=your-secret-key
JWT_EXPIRE=24h

# Server Configuration
PORT=8090
```

## Running the Application

### Using Docker Compose

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

### Manual Execution

Run each service individually:

```bash
# User Service
cd user-service
go run main.go start

# Product Service
cd product-service
go run main.go start

# Order Service
cd order-service
go run main.go start

# Payment Service
cd payment-service
go run main.go start

# Notification Service
cd notification-service
go run main.go start
```

### Using Makefile

If available, you can use make commands:

```bash
make run-user
make run-product
make run-order
make run-payment
make run-notification
```

## API Documentation

### Service Endpoints

#### User Service (http://localhost:8090)

-   `POST /api/v1/auth/register` - User registration
-   `POST /api/v1/auth/login` - User login
-   `GET /api/v1/users/profile` - Get user profile
-   `PUT /api/v1/users/profile` - Update user profile

#### Product Service (http://localhost:8082)

-   `GET /api/v1/products` - List all products
-   `GET /api/v1/products/:id` - Get product details
-   `POST /api/v1/products` - Create product (Admin)
-   `PUT /api/v1/products/:id` - Update product (Admin)
-   `DELETE /api/v1/products/:id` - Delete product (Admin)
-   `GET /api/v1/products/search` - Search products

#### Order Service (http://localhost:8083)

-   `POST /api/v1/orders` - Create order
-   `GET /api/v1/orders` - List user orders
-   `GET /api/v1/orders/:id` - Get order details
-   `PUT /api/v1/orders/:id/status` - Update order status
-   `DELETE /api/v1/orders/:id` - Cancel order

#### Payment Service (http://localhost:8084)

-   `POST /api/v1/payments` - Process payment
-   `GET /api/v1/payments/:id` - Get payment details
-   `PUT /api/v1/payments/:id/method` - Update payment method

#### Notification Service (http://localhost:8081)

-   `WS /ws` - WebSocket connection for real-time notifications
-   `GET /api/v1/notifications` - List user notifications

### Management Interfaces

-   **RabbitMQ Management**: http://localhost:15672 (guest/guest)
-   **Elasticsearch**: http://localhost:9200

## Database Migrations

Each service has its own database migrations in the `database/migrations` folder.

### Running Migrations

Migrations are typically run automatically on service startup, or you can run them manually:

```bash
# Example for order service
cd order-service
migrate -path database/migrations -database "postgresql://postgres:lokal@localhost:5434/order_service?sslmode=disable" up

# Rollback
migrate -path database/migrations -database "postgresql://postgres:lokal@localhost:5434/order_service?sslmode=disable" down
```

## Testing

### Load Testing

User service includes k6 load testing scripts:

```bash
cd user-service/test/k6
k6 run load-test.js
```

### Unit Testing

Run tests for each service:

```bash
cd user-service
go test ./... -v

cd ../product-service
go test ./... -v
```

## Project Structure

Each microservice follows a clean architecture pattern:

```
service-name/
├── cmd/                    # Application commands
│   ├── root.go
│   ├── start.go
│   └── worker-*.go        # Background workers
├── config/                 # Configuration
│   ├── config.go
│   ├── database.go
│   ├── rabbitmq.go
│   └── redis.go
├── database/               # Database related
│   ├── migrations/        # SQL migrations
│   └── seeds/             # Database seeds
├── internal/
│   ├── adapter/           # External adapters
│   │   ├── handlers/      # HTTP handlers
│   │   ├── repository/    # Database repositories
│   │   └── message/       # Message queue handlers
│   ├── core/
│   │   ├── domain/        # Domain entities
│   │   └── service/       # Business logic
│   └── app/               # Application setup
├── utils/                  # Utility functions
├── Dockerfile
├── go.mod
├── go.sum
└── main.go
```

## Message Queue Events

The services communicate via RabbitMQ with the following events:

-   **Order Created** → Payment Service, Notification Service
-   **Payment Processed** → Order Service, Notification Service
-   **Order Status Updated** → Notification Service
-   **Product Stock Updated** → Product Service
-   **Order Deleted** → Product Service (stock rollback)

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

[MIT License](LICENSE)
