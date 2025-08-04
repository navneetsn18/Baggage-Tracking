
# PRD: Baggage Tracking Microservice

**Version**: 1.0
**Date**: 2025-08-04

## 1. Overview

This document outlines the product requirements for the Baggage Tracking Microservice. This is the core service of the Realtime Baggage Tracking System, responsible for processing baggage scan events, updating baggage status, and providing realtime location data to users.

## 2. Purpose

The primary purpose of this microservice is to ingest baggage scan data, maintain a complete history of each bag's journey, and serve this information to end-users and other internal services. It acts as the single source of truth for baggage status and location.

## 3. Features

### 3.1. Baggage Status Updates
- **Description**: The service will update the status of a piece of baggage based on QR code scans performed by admin users.
- **Status Lifecycle**:
    - `CHECKED_IN`: Bag has been checked in.
    - `IN_TRANSIT`: Bag is moving through the airport system.
    - `LOADING`: Bag is being loaded onto the aircraft.
    - `ON_FLIGHT`: Bag is on the flight.
    - `UNLOADED`: Bag has been unloaded from the aircraft.
    - `DELIVERED`: Bag has been delivered to the baggage claim.
    - `DELAYED`: Bag is delayed.
    - `LOST`: Bag is considered lost.
- **Acceptance Criteria**:
    - Each scan event must update the baggage status accordingly.
    - The service must validate that the scan location is logical (e.g., a bag cannot be `DELIVERED` before it is `UNLOADED`).

### 3.2. Realtime Location Tracking
- **Description**: Provide live location updates of baggage to the frontend.
- **Technology**: WebSockets or Server-Sent Events (SSE).
- **Acceptance Criteria**:
    - The frontend should be able to subscribe to location updates for a specific baggage ID.
    - Location updates should be pushed to the frontend in realtime (<1s latency).

### 3.3. Baggage History Log
- **Description**: Maintain a complete, timestamped log of all scan events for each piece of baggage.
- **Acceptance Criteria**:
    - Every scan event must be recorded with a timestamp, location, and the ID of the admin who performed the scan.
    - The history log must be retrievable via the API.

## 4. API Endpoints

### 4.1. `POST /tracking/register`
- **Description**: Creates a new baggage record. Called by the Admin Operations service.
- **Request Body**:
    ```json
    {
      "pnr": "ABC123",
      "last_name": "Smith",
      "flight_id": "AA789"
    }
    ```
- **Success Response (201 Created)**:
    ```json
    {
      "baggage_id": "BJS123"
    }
    ```
- **Error Responses**:
    - `400 Bad Request`: Invalid data.

### 4.2. `POST /tracking/scan` (Admin)
- **Description**: Receives a scan event from an admin user to update baggage status.
- **Request Body**:
    ```json
    {
      "baggage_id": "BJS123",
      "scan_location": "Terminal A-Security",
      "admin_id": "..."
    }
    ```
- **Success Response (202 OK)**: The request is accepted for processing.
- **Error Responses**:
    - `400 Bad Request`: Invalid data.
    - `404 Not Found`: Baggage ID not found.

### 4.3. `GET /tracking/{pnr_or_baggage_id}` (User)
- **Description**: Returns the current status and location history of a piece of baggage.
- **Success Response (200 OK)**:
    ```json
    {
      "baggage_id": "BJS123",
      "pnr": "ABC123",
      "status": "IN_TRANSIT",
      "history": [
        {
          "location": "Check-in Counter A",
          "timestamp": "2025-08-04T10:00:00Z"
        },
        {
          "location": "Terminal A-Security",
          "timestamp": "2025-08-04T10:15:00Z"
        }
      ]
    }
    ```
- **Error Responses**:
    - `404 Not Found`: PNR or baggage ID not found.

### 4.4. `GET /tracking/map/{baggage_id}`
- **Description**: Returns GeoJSON coordinates for map visualization.
- **Success Response (200 OK)**:
    ```json
    {
      "type": "Feature",
      "properties": {
        "baggage_id": "BJS123"
      },
      "geometry": {
        "type": "Point",
        "coordinates": [-74.0060, 40.7128]
      }
    }
    ```
- **Error Responses**:
    - `404 Not Found`: Baggage ID not found.

## 5. Database Schema

### 5.1. `Baggage` Table
| Column      | Type          | Constraints           | Description                                   |
|-------------|---------------|-----------------------|-----------------------------------------------|
| `id`        | `VARCHAR(255)`| Primary Key, Not Null | Unique identifier for the baggage (e.g., BJS123). |
| `pnr`       | `VARCHAR(255)`| Not Null              | Passenger Name Record.                        |
| `last_name` | `VARCHAR(255)`| Not Null              | Passenger's last name.                        |
| `flight_id` | `VARCHAR(255)`| Not Null              | The flight the baggage is associated with.    |
| `status`    | `VARCHAR(50)` | Not Null              | Current status of the baggage.                |

### 5.2. `ScanEvents` Table
| Column      | Type          | Constraints                           | Description                                   |
|-------------|---------------|---------------------------------------|-----------------------------------------------|
| `id`        | `UUID`        | Primary Key, Not Null                 | Unique identifier for the scan event.         |
| `baggage_id`| `VARCHAR(255)`| Foreign Key to `Baggage(id)`, Not Null| The ID of the scanned baggage.                |
| `location`  | `VARCHAR(255)`| Not Null                              | The location where the scan occurred.         |
| `timestamp` | `TIMESTAMPTZ` | Not Null                              | The time of the scan.                         |
| `admin_id`  | `UUID`        | Not Null                              | The ID of the admin who performed the scan.   |

### 5.3. `BaggageLocation` Table (for map visualization)
| Column      | Type          | Constraints                           | Description                                   |
|-------------|---------------|---------------------------------------|-----------------------------------------------|
| `baggage_id`| `VARCHAR(255)`| Foreign Key to `Baggage(id)`, Not Null| The ID of the baggage.                        |
| `latitude`  | `DECIMAL(9,6)`| Not Null                              | Latitude of the baggage.                      |
| `longitude` | `DECIMAL(9,6)`| Not Null                              | Longitude of the baggage.                     |
| `timestamp` | `TIMESTAMPTZ` | Not Null                              | The time of the location update.              |

## 6. Non-Functional Requirements

- **Configuration**: All external configuration (e.g., database connection strings, Kafka broker addresses) will be managed via **environment variables**.
- **Secrets Management**: All secrets (e.g., database credentials) will be managed via **environment variables**.
- **Database Schema**: Database schema will be created manually or via simple SQL scripts for the hackathon.
- **Logging**: Basic application logs will be output to standard output.
- **Realtime**: Status updates should be processed and propagated to the frontend in under 1 second.
- **Data Integrity**: The system must ensure that the baggage status follows a logical progression.
- **Scalability**: Not a primary concern for the hackathon MVP.

## 5. Database Schema

### 5.1. `Baggage` Table
| Column      | Type          | Constraints           | Description                                   |
|-------------|---------------|-----------------------|-----------------------------------------------|
| `id`        | `VARCHAR(255)`| Primary Key, Not Null | Unique identifier for the baggage (e.g., BJS123). |
| `pnr`       | `VARCHAR(255)`| Not Null              | Passenger Name Record.                        |
| `last_name` | `VARCHAR(255)`| Not Null              | Passenger's last name.                        |
| `flight_id` | `VARCHAR(255)`| Not Null              | The flight the baggage is associated with.    |
| `status`    | `VARCHAR(50)` | Not Null              | Current status of the baggage.                |

### 5.2. `ScanEvents` Table
| Column      | Type          | Constraints                           | Description                                   |
|-------------|---------------|---------------------------------------|-----------------------------------------------|
| `id`        | `UUID`        | Primary Key, Not Null                 | Unique identifier for the scan event.         |
| `baggage_id`| `VARCHAR(255)`| Foreign Key to `Baggage(id)`, Not Null| The ID of the scanned baggage.                |
| `location`  | `VARCHAR(255)`| Not Null                              | The location where the scan occurred.         |
| `timestamp` | `TIMESTAMPTZ` | Not Null                              | The time of the scan.                         |
| `admin_id`  | `UUID`        | Not Null                              | The ID of the admin who performed the scan.   |

### 5.3. `BaggageLocation` Table (for map visualization)
| Column      | Type          | Constraints                           | Description                                   |
|-------------|---------------|---------------------------------------|-----------------------------------------------|
| `baggage_id`| `VARCHAR(255)`| Foreign Key to `Baggage(id)`, Not Null| The ID of the baggage.                        |
| `latitude`  | `DECIMAL(9,6)`| Not Null                              | Latitude of the baggage.                      |
| `longitude` | `DECIMAL(9,6)`| Not Null                              | Longitude of the baggage.                     |
| `timestamp` | `TIMESTAMPTZ` | Not Null                              | The time of the location update.              |

## 6. Non-Functional Requirements

- **Configuration**: All external configuration, including database connection strings and Kafka broker addresses, must be loaded from **AWS Systems Manager Parameter Store** on startup.
- **Secrets Management**: All secrets, including database credentials, must be securely fetched from **AWS Secrets Manager**.
- **Database Migrations**: All schema changes must be managed and version-controlled using **Flyway**.
- **API Documentation**: The service must generate a standard **OpenAPI 3 specification** from its code.
- **Logging**: The service must use **Structured Logging** (JSON format) and must propagate the `correlation-id` received in HTTP headers to all log messages and downstream Kafka messages.
- **Standardized Error Handling**: All API responses must adhere to the standardized error format defined in the Architecture Overview.
- **Service-Level Rate Limiting**: Critical internal endpoints or resource-intensive operations will implement their own rate limiting mechanisms.
- **Realtime**: Status updates should be processed and propagated to the frontend in under 1 second.
- **Data Integrity**: The system must ensure that the baggage status follows a logical progression.
- **Scalability**: The service must be able to handle a high volume of scan events, especially during peak travel times.
