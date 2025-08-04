
# PRD: Admin Operations Microservice

**Version**: 1.0
**Date**: 2025-08-04

## 1. Overview

This document outlines the product requirements for the Admin Operations Microservice. This service provides the tools and interfaces for airport staff to manage and track baggage.

## 2. Purpose

The primary purpose of this microservice is to empower admin users with the functionality they need to perform their duties, including scanning QR codes, monitoring baggage status, and managing baggage assignments.

## 3. Features

### 3.1. QR Code Scanning
- **Description**: Admins will use a mobile-responsive web interface to scan QR codes on baggage tags.
- **Acceptance Criteria**:
    - The scanning interface must be fast and reliable.
    - A successful scan must trigger a call to the Baggage Tracking microservice to update the baggage status.

### 3.2. Admin Dashboard
- **Description**: A dashboard for admins to monitor baggage handling operations.
- **Features**:
    - **Scan Analytics**: View the number of scans per hour, per location, etc.
    - **Misplaced Baggage Alerts**: A list of baggage that has an abnormal status (e.g., off-route, delayed).
- **Acceptance Criteria**:
    - The dashboard must provide a clear and concise overview of the baggage handling process.
    - The misplaced baggage list must be updated in realtime.

### 3.3. Baggage Registration & QR Code Generation
- **Description**: Admins can register new baggage. This service orchestrates the registration process by fetching flight details and instructing the Baggage Tracking service to create the record.
- **Workflow**:
    1. Receives PNR and Last Name from the frontend.
    2. Calls the `User Profile & Flight Service` (`GET /flights?pnr={pnr}`) to retrieve the `flight_id`.
    3. Calls the `Baggage Tracking Service` (`POST /tracking/register`) with PNR, Last Name, and `flight_id`.
    4. Returns the newly generated `baggage_id` to the frontend for QR code generation.
- **Acceptance Criteria**:
    - The service must successfully coordinate with both the User Profile and Baggage Tracking services.
    - Must handle errors gracefully if the PNR is not found or the tracking service fails.

### 3.4. Baggage Assignment
- **Description**: Admins can assign baggage to specific flights or carousels.
- **Acceptance Criteria**:
    - The service must provide an interface for assigning a baggage ID to a flight ID or carousel number.
    - This information must be persisted and available to other services.

### 3.5. Reverse Baggage Lookup
- **Description**: Admins can look up user and flight details using a baggage ID.
- **Acceptance Criteria**:
    - The service must accept a baggage ID and return the associated PNR, last name, and flight details.

### 3.6. Flight-Centric Baggage View
- **Description**: Admins can view all baggage associated with a specific flight.
- **Acceptance Criteria**:
    - The service must provide an endpoint to retrieve a list of all baggage and their statuses for a given `flight_id`.

### 3.7. Baggage Re-routing
- **Description**: Admins can re-route a piece of baggage to a different flight.
- **Acceptance Criteria**:
    - The service must provide an endpoint to update the `flight_id` for a given `baggage_id`.
    - This change must be propagated to the Baggage Tracking service.

### 3.8. Lost Baggage Protocol
- **Description**: Admins can flag a piece of baggage as lost, initiating a lost baggage workflow.
- **Acceptance Criteria**:
    - The service must provide an endpoint to update the baggage status to `LOST`.
    - This should trigger appropriate notifications and potentially integrate with a lost-and-found system.

## 4. API Endpoints

### 4.1. `POST /admin/register-baggage`
- **Description**: Registers a new piece of baggage. This endpoint acts as a Saga orchestrator.
- **Request Body**:
    ```json
    {
      "pnr": "ABC123",
      "last_name": "Smith"
    }
    ```
- **Success Response (201 Created)**:
    ```json
    {
      "baggage_id": "BJS123",
      "qr_code_data": "BJS123"
    }
    ```
- **Error Responses**:
    - `400 Bad Request`: Invalid data.
    - `404 Not Found`: PNR not found.

### 4.2. `POST /admin/scan`
- **Description**: Receives a QR code scan from an admin user. This endpoint will invoke the Baggage Tracking service.
- **Request Body**:
    ```json
    {
      "baggage_id": "BJS123",
      "scan_location": "Terminal A-Security",
      "admin_id": "..."
    }
    ```
- **Success Response (202 Accepted)**: The scan has been accepted for processing.
- **Error Responses**:
    - `400 Bad Request`: Invalid data.

### 4.3. `GET /admin/baggage/{baggage_id}`
- **Description**: Retrieves user and flight details for a given baggage ID.
- **Success Response (200 OK)**:
    ```json
    {
      "pnr": "ABC123",
      "last_name": "Smith",
      "flight_id": "AA789"
    }
    ```
- **Error Responses**:
    - `404 Not Found`: Baggage ID not found.

### 4.4. `GET /admin/flights/{flight_id}/baggage`
- **Description**: Retrieves all baggage for a given flight.
- **Success Response (200 OK)**:
    ```json
    [
      {
        "baggage_id": "BJS123",
        "status": "IN_TRANSIT"
      },
      {
        "baggage_id": "BJS456",
        "status": "LOADING"
      }
    ]
    ```
- **Error Responses**:
    - `404 Not Found`: Flight ID not found.

### 4.5. `PUT /admin/baggage/{baggage_id}/reroute`
- **Description**: Re-routes a piece of baggage to a new flight.
- **Request Body**:
    ```json
    {
      "new_flight_id": "UA456"
    }
    ```
- **Success Response (200 OK)**.
- **Error Responses**:
    - `400 Bad Request`: Invalid data.
    - `404 Not Found`: Baggage ID not found.

### 4.6. `POST /admin/baggage/{baggage_id}/report-lost`
- **Description**: Reports a piece of baggage as lost.
- **Success Response (200 OK)**.
- **Error Responses**:
    - `404 Not Found`: Baggage ID not found.

### 4.7. `GET /admin/misplaced`
- **Description**: Returns a list of baggage with an abnormal status.
- **Success Response (200 OK)**:
    ```json
    [
      {
        "baggage_id": "BJS456",
        "last_known_location": "Terminal B-Security",
        "status": "OFF_ROUTE"
      }
    ]
    ```

### 4.8. `POST /admin/assign`
- **Description**: Assigns a piece of baggage to a flight or carousel.
- **Request Body**:
    ```json
    {
      "baggage_id": "BJS123",
      "assign_to": "flight/AA789" 
    }
    ```
- **Success Response (200 OK)**.
- **Error Responses**:
    - `400 Bad Request`: Invalid data.
    - `404 Not Found`: Baggage ID not found.

## 5. Non-Functional Requirements

- **Configuration**: All external configuration, including database connection strings and Kafka broker addresses, must be loaded from **AWS Systems Manager Parameter Store** on startup.
- **Secrets Management**: All secrets, including passwords and API keys, must be securely fetched from **AWS Secrets Manager**.
- **API Documentation**: The service must generate a standard **OpenAPI 3 specification** from its code, providing a live, interactive API console for developers.
- **Logging**: The service must use **Structured Logging** (JSON format) and must propagate the `correlation-id` received in HTTP headers to all log messages and downstream API calls.
- **Usability**: The admin interfaces must be intuitive and easy to use, especially the mobile scanning interface.
- **Performance**: The admin dashboard must load quickly, even with a large amount of data.
- **Security**: All admin endpoints must be protected and only accessible to users with the `ADMIN` role and the required scopes.
