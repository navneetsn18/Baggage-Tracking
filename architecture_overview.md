# Architecture Overview (Hackathon Edition)

**Version**: 1.0 (Hackathon)
**Date**: 2025-08-04

## 1. API Gateway (Conceptual)

For the hackathon, the API Gateway will be conceptual, primarily handled by direct service calls from the frontend or a simple Nginx reverse proxy if time permits. Its core responsibilities are simplified:

### 1.1. Responsibilities

- **Request Routing**: Direct calls from frontend to microservices.
- **Authentication & Authorization**: Frontend will send JWTs, and microservices will validate them directly.
- **CORS Management**: Handled directly within each Spring Boot microservice (as shown in `deployment_plan.md`).

### 1.2. Request Flow

1. The Next.js frontend sends an API request directly to the relevant microservice.
2. The microservice validates the JWT (if present) and processes the request.
3. The microservice returns a response to the frontend.

## 2. Microservice Interaction

Microservices will interact directly via HTTP calls or Kafka messages. Complex distributed transaction patterns like Saga are out of scope for this hackathon.

### 2.1. Baggage Registration Flow (Simplified)

1. Frontend sends `POST /admin/register-baggage` to Admin Operations Service.
2. Admin Operations Service calls `GET /flights?pnr={pnr}` on User Profile & Flight Service.
3. Admin Operations Service calls `POST /tracking/register` on Baggage Tracking Service.
4. Baggage Tracking Service returns `baggage_id`.
5. Admin Operations Service returns `baggage_id` to frontend.