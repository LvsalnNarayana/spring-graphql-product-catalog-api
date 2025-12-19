
# Spring-Graphql-Product-Catalog-Api

## Overview

This project is a **multi-microservice demonstration** of **GraphQL** in Spring Boot 3.x using **Spring GraphQL** (formerly GraphQL Java Spring). It showcases how GraphQL solves common REST over-fetching/under-fetching problems by allowing clients to request exactly the data they need.

The focus is on building a performant, flexible product catalog API with advanced features like **batching** (DataLoader), **schema stitching**, and **custom resolvers**.

## Real-World Scenario (Simulated)

In e-commerce platforms like **eBay** or **Amazon**:
- Mobile and web clients have different data requirements for the same product page.
- REST APIs often return too much (over-fetching) or require multiple round-trips (under-fetching).
- High-traffic product detail pages need efficient data loading to avoid N+1 query problems.

We simulate a product catalog where clients can flexibly query products, reviews, recommendations, and related items in a single request — with server-side optimizations for performance.

## Microservices Involved

| Service                       | Responsibility                                                                 | Port  |
|-------------------------------|--------------------------------------------------------------------------------|-------|
| **eureka-server**             | Service discovery (Netflix Eureka)                                             | 8761  |
| **product-service**           | Core product data (name, price, description, category)                         | 8081  |
| **review-service**            | Product reviews and ratings                                                    | 8082  |
| **recommendation-service**    | Related/similar products (simulated ML recommendations)                        | 8083  |
| **api-gateway** (optional)    | GraphQL gateway aggregating schemas from all services (schema stitching demo) | 8080  |

The main GraphQL endpoint is in **product-service** (or gateway), with federated resolvers calling downstream services.

## Tech Stack

- Spring Boot 3.x
- Spring GraphQL (spring-boot-starter-graphql)
- GraphQL Java Tools + DGS (Netflix DataLoader optional)
- Spring Data JPA + H2/PostgreSQL
- Spring Cloud OpenFeign (for inter-service calls)
- Spring Cloud Netflix Eureka
- Project Lombok
- Maven (multi-module)
- Docker & Docker Compose
- GraphiQL/GraphQL Playground (embedded UI)

## Docker Containers

```yaml
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: catalog
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"

  eureka-server:
    build: ./eureka-server
    ports:
      - "8761:8761"

  product-service:
    build: ./product-service
    depends_on:
      - postgres
      - eureka-server
    ports:
      - "8081:8081"

  review-service:
    build: ./review-service
    depends_on:
      - postgres
      - eureka-server
    ports:
      - "8082:8082"

  recommendation-service:
    build: ./recommendation-service
    depends_on:
      - eureka-server
    ports:
      - "8083:8083"
```

Run with: `docker-compose up --build`

## GraphQL Features Implemented

| Feature                  | Implementation Details                                                  |
|--------------------------|-------------------------------------------------------------------------|
| **Schema First**         | `.graphqls` files defining types, queries, mutations                    |
| **Flexible Queries**     | Clients request only needed fields (e.g., name + price, or full details)|
| **Batching (N+1 Fix)**   | DataLoader for reviews and recommendations                             |
| **Custom Resolvers**     | Field-level resolvers calling Feign clients to other services           |
| **Schema Stitching**     | Optional gateway combines schemas (demo included)                       |
| **Error Handling**       | Custom GraphQL exceptions with extensions                               |

## Key Features

- Single GraphQL endpoint for product catalog
- Efficient batch loading of reviews/recommendations
- Inter-service data fetching with Feign + Eureka discovery
- Embedded GraphQL IDE (GraphiQL) at `/graphiql`
- Custom scalars and input types
- Query depth limiting (security)
- Support for queries, mutations, and subscriptions (basic)
- Traceable via response headers

## Expected Endpoints

### GraphQL Endpoint (`http://localhost:8081/graphql`)

- POST `/graphql` — main query endpoint
- GET `/graphiql` — interactive IDE (browser-based)

### Sample Queries

**Simple Product List**
```graphql
query {
  products(first: 5) {
    edges {
      node {
        id
        name
        price
      }
    }
  }
}
```

**Full Product with Reviews & Recommendations (no over-fetching)**
```graphql
query {
  product(id: "1") {
    id
    name
    description
    price
    reviews(first: 3) {
      edges {
        node {
          rating
          comment
          author
        }
      }
    }
    recommendations(first: 4) {
      id
      name
      price
    }
  }
}
```

**Mutation: Add Review**
```graphql
mutation {
  addReview(input: {productId: "1", rating: 5, comment: "Great!"}) {
    id
    rating
  }
}
```

## Architecture Overview

```
Client (Web/Mobile)
   ↓ (single request)
GraphQL Endpoint (product-service or gateway)
   ↓
Schema Resolvers
   ├── Product (local JPA)
   ├── Reviews → Feign → review-service + DataLoader batch
   └── Recommendations → Feign → recommendation-service + DataLoader
   ↓
PostgreSQL (shared or separate)
```

**N+1 Prevention**:
- Multiple product reviews requested → batched into single call to review-service

## How to Run

1. Clone repository
2. Start Docker: `docker-compose up --build`
3. Open GraphiQL: `http://localhost:8081/graphiql`
4. Explore schema in "Docs" tab
5. Run sample queries above
6. Observe batching in logs (single call for multiple reviews)

## Testing GraphQL

1. Query products without reviews → fast
2. Query 10 products with reviews → DataLoader batches → 1 call to review-service
3. Compare vs REST: fewer round-trips, exact data
4. Use different clients (mobile vs desktop) → same endpoint, different fields

## Skills Demonstrated

- GraphQL schema design (SDL)
- Spring GraphQL resolvers and controllers
- Solving N+1 with DataLoader
- Inter-service GraphQL data fetching
- Schema stitching/federation basics
- Flexible client-driven queries
- Performance optimization in APIs

## Future Extensions

- Full Apollo Federation
- Subscriptions for real-time review updates
- Rate limiting and persisted queries
- Authentication/Authorization (GraphQL @PreAuthorize)
- GraphQL over WebSocket
- Caching with Redis

