
# PRD: User Profile & Flight Microservice

**Version**: 1.0
**Date**: 2025-08-04

## 1. Overview

This document outlines the product requirements for the User Profile and Flight Microservice. This service is responsible for managing user data, flight information, and the relationship between users, flights, and baggage.

## 2. Purpose

The primary purpose of this microservice is to provide a centralized location for user and flight data. It will serve as the backend for the user-facing portal, providing all the necessary information for a user to track their baggage and view their flight details.

## 3. Features

### 3.1. User Profiles
- **Description**: The service will manage user profiles, including contact information and notification preferences.
- **Acceptance Criteria**:
    - A user's profile must be retrievable via an API endpoint.
    - Users must be able to update their contact information and notification preferences.

### 3.2. Flight Information
- **Description**: The service will store and provide flight information, including routes and schedules. For the hackathon, this may be simplified to mock data or a very basic third-party integration.
- **Integration**: This service will integrate with third-party APIs (e.g., AviationStack) to pull in realtime flight data, or use mock data.
- **Acceptance Criteria**:
    - The service must be able to retrieve flight details for a given PNR.
    - The service must be able to provide the flight path as a GeoJSON for map overlay.

### 3.3. Baggage-Flight Mapping
- **Description**: The service will maintain the mapping between a user's baggage and their flight.
- **Acceptance Criteria**:
    - The service must be able to associate a baggage ID with a flight ID.
    - This mapping must be retrievable via the API.

## 4. API Endpoints

### 4.1. `GET /user/{id}`
- **Description**: Retrieves the profile for a given user.
- **Success Response (200 OK)**:
    ```json
    {
      "id": "...",
      "email": "user@example.com",
      "notification_preferences": {
        "email": true,
        "sms": false
      }
    }
    ```
- **Error Responses**:
    - `404 Not Found`: User not found.

### 4.2. `GET /flights?pnr={pnr}`
- **Description**: Retrieves flight details and associated baggage status for a given PNR.
- **Success Response (200 OK)**:
    ```json
    {
      "pnr": "ABC123",
      "flight_id": "AA789",
      "departure_airport": "JFK",
      "arrival_airport": "LAX",
      "departure_time": "2025-08-04T12:00:00Z",
      "arrival_time": "2025-08-04T15:00:00Z",
      "baggage_status": "IN_TRANSIT"
    }
    ```
- **Error Responses**:
    - `404 Not Found`: PNR not found.

### 4.3. `GET /flights/route/{flight_id}`
- **Description**: Retrieves the flight path as a GeoJSON for map overlay.
- **Success Response (200 OK)**:
    ```json
    {
      "type": "Feature",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [-74.0060, 40.7128],
          [-118.2437, 34.0522]
        ]
      }
    }
    ```
- **Error Responses**:
    - `404 Not Found`: Flight ID not found.

## 5. Non-Functional Requirements

- **Configuration**: All external configuration (e.g., database connection strings, third-party API keys) will be managed via **environment variables**.
- **Secrets Management**: All secrets (e.g., database credentials, third-party API keys) will be managed via **environment variables**.
- **Database Schema**: Database schema will be created manually or via simple SQL scripts for the hackathon.
- **Logging**: Basic application logs will be output to standard output.
- **Data Privacy**: All user data must be handled in accordance with GDPR/CCPA regulations (simplified for hackathon).
- **Third-Party Integration**: The integration with third-party flight data providers may be simplified to mock data for the hackathon.
- **Performance**: Basic performance is expected for a hackathon MVP.

## 5. Non-Functional Requirements

- **Configuration**: All external configuration, including database connection strings and third-party API keys, must be loaded from **AWS Systems Manager Parameter Store** on startup.
- **Secrets Management**: All secrets, including database credentials and third-party API keys, must be securely fetched from **AWS Secrets Manager**.
- **Database Migrations**: All schema changes must be managed and version-controlled using **Flyway**.
- **API Documentation**: The service must generate a standard **OpenAPI 3 specification** from its code.
- **Logging**: The service must use **Structured Logging** (JSON format) and must propagate the `correlation-id` received in HTTP headers to all log messages.
- **Standardized Error Handling**: All API responses must adhere to the standardized error format defined in the Architecture Overview.
- **Service-Level Rate Limiting**: Critical internal endpoints or resource-intensive operations will implement their own rate limiting mechanisms.
- **Data Privacy**: All user data must be handled in accordance with GDPR/CCPA regulations.
- **Third-Party Integration**: The integration with third-party flight data providers must be reliable and handle potential API failures gracefully.
- **Performance**: The API endpoints must respond quickly to provide a smooth user experience.
