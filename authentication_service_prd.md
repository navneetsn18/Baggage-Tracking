
# PRD: Authentication & Authorization Microservice

**Version**: 1.0
**Date**: 2025-08-04

## 1. Overview

This document outlines the product requirements for the Authentication and Authorization Microservice. This service is responsible for managing user and admin identities, securing access to the platform, and providing authentication tokens for inter-service communication.

## 2. Purpose

The core purpose of this microservice is to provide a robust and secure authentication and authorization layer for the entire Realtime Baggage Tracking System. It will handle user login, token generation, and role-based access control (RBAC) to ensure that users and system components only have access to the resources they are permitted to use.

## 3. Features

### 3.1. JWT-Based Authentication
- **Description**: The service will implement JSON Web Token (JWT) based authentication compliant with the OAuth 2.0 standard.
- **Acceptance Criteria**:
    - Successful login returns a JWT access token and a refresh token.
    - The JWT payload must contain the user's ID, email, and role.
    - Tokens must have a defined expiration time (e.g., 15 minutes for access tokens, 7 days for refresh tokens).

### 3.2. Role & Scope Management
- **Description**: The service will manage different user roles and fine-grained scopes to control access to various parts of the system.
- **Roles & Scopes**:
    - `USER`: Standard passenger user. Scopes: `baggage:read`
    - `ADMIN`: Airport staff with sub-roles for specific functions.
        - `CHECKIN`: Staff at the check-in counter. Scopes: `baggage:register`, `baggage:read`
        - `SECURITY`: Security personnel. Scopes: `baggage:scan`, `baggage:read`
        - `GATE`: Gate agents. Scopes: `baggage:scan`, `baggage:read`, `flight:read`
        - `CARGO`: Cargo and baggage handling staff. Scopes: `baggage:scan`, `baggage:read`, `baggage:reroute`
        - `BAGGAGE_CLAIM`: Staff at the baggage claim area. Scopes: `baggage:scan`, `baggage:read`, `baggage:report_lost`
- **Acceptance Criteria**:
    - The user's role and a list of their assigned scopes must be included in the JWT claims.
    - The system must be able to differentiate between `USER` and `ADMIN` roles and their respective sub-roles and scopes.

### 3.3. Endpoint Security
- **Description**: The service will provide a mechanism for other microservices to validate JWTs and secure their endpoints.
- **Acceptance Criteria**:
    - An endpoint must be available to validate a given JWT.

## 4. API Endpoints

### 4.1. `POST /auth/login`
- **Description**: Authenticates a user and returns a JWT.
- **Request Body**:
    ```json
    {
      "email": "user@example.com",
      "password": "password123"
    }
    ```
- **Success Response (200 OK)**:
    ```json
    {
      "access_token": "...",
      "refresh_token": "...",
      "token_type": "Bearer",
      "expires_in": 900
    }
    ```
- **Error Responses**:
    - `401 Unauthorized`: Invalid credentials.
    - `400 Bad Request`: Missing fields.

### 4.2. `POST /auth/validate`
- **Description**: Validates a JWT. Intended for use by other microservices.
- **Request Body**:
    ```json
    {
      "token": "..."
    }
    ```
- **Success Response (200 OK)**:
    ```json
    {
      "valid": true,
      "claims": {
        "user_id": "...",
        "email": "...",
        "role": "..."
      }
    }
    ```
- **Error Responses**:
    - `401 Unauthorized`: Invalid or expired token.

### 4.3. `POST /auth/refresh`
- **Description**: Issues a new access token using a refresh token.
- **Request Body**:
    ```json
    {
      "refresh_token": "..."
    }
    ```
- **Success Response (200 OK)**:
    ```json
    {
      "access_token": "...",
      "token_type": "Bearer",
      "expires_in": 900
    }
    ```
- **Error Responses**:
    - `401 Unauthorized`: Invalid or expired refresh token.

## 5. Database Schema

### 5.1. `Users` Table
| Column          | Type          | Constraints              | Description                               |
|-----------------|---------------|--------------------------|-------------------------------------------|
| `id`            | `UUID`        | Primary Key, Not Null    | Unique identifier for the user.           |
| `email`         | `VARCHAR(255)`| Unique, Not Null         | User's email address.                     |
| `password_hash` | `VARCHAR(255)`| Not Null                 | Hashed password.                          |
| `role`          | `VARCHAR(50)` | Not Null                 | User's role (e.g., `USER`, `ADMIN`).      |
| `location`      | `VARCHAR(255)`| Nullable                 | Physical location for admin users.        |

### 5.2. `AdminLocations` Table
| Column        | Type          | Constraints                               | Description                               |
|---------------|---------------|-------------------------------------------|-------------------------------------------|
| `admin_id`    | `UUID`        | Foreign Key to `Users(id)`, Not Null      | The ID of the admin user.                 |
| `airport_zone`| `VARCHAR(255)`| Not Null                                  | The specific zone the admin is assigned to. |

## 6. Non-Functional Requirements

- **Configuration**: All external configuration (e.g., database connection strings) will be managed via **environment variables**.
- **Secrets Management**: All secrets (e.g., passwords, JWT signing keys) will be managed via **environment variables**.
- **Database Schema**: Database schema will be created manually or via simple SQL scripts for the hackathon.
- **Logging**: Basic application logs will be output to standard output.
- **Security**:
    - Passwords must be hashed using a strong algorithm (e.g., bcrypt).
    - All communication must be over TLS 1.3.
- **Performance**: Basic performance is expected for a hackathon MVP.
- **Scalability**: Not a primary concern for the hackathon MVP.
