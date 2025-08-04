
# PRD: Notification Microservice

**Version**: 1.0
**Date**: 2025-08-04

## 1. Overview

This document outlines the product requirements for the Notification Microservice. This service is responsible for sending multi-channel alerts to users about changes in their baggage status.

## 2. Purpose

The primary purpose of this microservice is to keep users informed about the status of their baggage in realtime. It will consume events from a message queue, determine the user's notification preferences, and dispatch alerts through the appropriate channels.

## 3. Features

### 3.1. Multi-Channel Notifications
- **Description**: The service will support sending notifications via multiple channels.
- **Channels**:
    - Email (via SendGrid)
    - SMS (via Twilio)
    - WhatsApp (via Twilio API)
    ### 3.1. Multi-Channel Notifications
- **Description**: The service will support sending notifications via multiple channels. For the hackathon, this may be simplified to logging notifications or using free-tier/mock APIs.
- **Channels**:
    - Email (via SendGrid - if free tier available, else mock)
    - SMS (via Twilio - if free tier available, else mock)
    - WhatsApp (via Twilio API - if free tier available, else mock)
    - Push Notifications (via Firebase - if free tier available, else mock)
- **Acceptance Criteria**:
    - The service must be able to send notifications to any of the supported channels (or log them if external APIs are not used).
    - The integration with each third-party service must be robust and handle potential failures.

### 3.2. User-Configurable Preferences
- **Description**: Users can configure their notification preferences, including which channels to use and for which events they want to be notified.
- **Preferences**:
    - `NOTIFY_ON_ALL_SCANS`: Receive an alert for every scan event.
    - `NOTIFY_ON_DELAY`: Receive an alert only if the baggage is delayed.
    - `NOTIFY_ON_DELIVERY`: Receive an alert when the baggage is delivered.
- **Acceptance Criteria**:
    - The service must respect the user's notification preferences.
    - A user's preferences must be retrievable via an API endpoint.

### 3.3. Kafka Consumer
- **Description**: The service will listen for baggage events on Kafka topics.
- **Topics**:
    - `scan_events`: For regular baggage scan updates.
    - `delay_risk_events`: For proactive notifications about potential delays (if implemented).
- **Acceptance Criteria**:
    - The service must be able to consume and process messages from all relevant topics.
    - The consumer must be resilient and able to handle message processing failures.

## 4. API Endpoints

### 4.1. `POST /notify/send`
- **Description**: This endpoint is triggered by the Kafka listener to send a notification.
- **Request Body**:
    ```json
    {
      "user_id": "...",
      "baggage_id": "BJS123",
      "event_type": "DELAYED",
      "message": "Your bag is delayed by 20 mins."
    }
    ```
- **Success Response (202 Accepted)**: The notification request has been queued for sending.
- **Error Responses**:
    - `400 Bad Request`: Invalid data.

### 4.2. `GET /notify/preferences/{user_id}`
- **Description**: Retrieves the notification preferences for a given user.
- **Success Response (200 OK)**:
    ```json
    {
      "user_id": "...",
      "preferences": {
        "email": {
          "enabled": true,
          "address": "user@example.com"
        },
        "sms": {
          "enabled": false
        }
      }
    }
    ```
- **Error Responses**:
    - `404 Not Found`: User not found.

## 5. Workflow

1. A baggage scan event is published to the `scan_events` Kafka topic by the Baggage Tracking microservice.
2. The Notification microservice consumes the event.
3. It retrieves the user's notification preferences from the User Profile microservice.
4. It dispatches alerts via the user's preferred channels (e.g., SendGrid for email, Twilio for SMS), or logs them.

## 6. Non-Functional Requirements

- **Configuration**: All external configuration (e.g., Kafka broker addresses, API keys for external services) will be managed via **environment variables**.
- **Secrets Management**: All secrets (e.g., API keys) will be managed via **environment variables**.
- **Logging**: Basic application logs will be output to standard output.
- **Timeliness**: Notifications should be sent within 5 seconds of the event being consumed.
- **Reliability**: The service should aim for at-least-once delivery of notifications.
- **Scalability**: Not a primary concern for the hackathon MVP.
- **Acceptance Criteria**:
    - The service must be able to send notifications to any of the supported channels.
    - The integration with each third-party service must be robust and handle potential failures.

### 3.2. User-Configurable Preferences
- **Description**: Users can configure their notification preferences, including which channels to use and for which events they want to be notified.
- **Preferences**:
    - `NOTIFY_ON_ALL_SCANS`: Receive an alert for every scan event.
    - `NOTIFY_ON_DELAY`: Receive an alert only if the baggage is delayed.
    - `NOTIFY_ON_DELIVERY`: Receive an alert when the baggage is delivered.
- **Acceptance Criteria**:
    - The service must respect the user's notification preferences.
    - A user's preferences must be retrievable via an API endpoint.

### 3.3. Kafka Consumer
- **Description**: The service will listen for baggage events on Kafka topics.
- **Topics**:
    - `scan_events`: For regular baggage scan updates.
    - `delay_risk_events`: For proactive notifications about potential delays.
- **Acceptance Criteria**:
    - The service must be able to consume and process messages from all relevant topics.
    - The consumer must be resilient and able to handle message processing failures.

## 4. API Endpoints

### 4.1. `POST /notify/send`
- **Description**: This endpoint is triggered by the Kafka listener to send a notification.
- **Request Body**:
    ```json
    {
      "user_id": "...",
      "baggage_id": "BJS123",
      "event_type": "DELAYED",
      "message": "Your bag is delayed by 20 mins."
    }
    ```
- **Success Response (202 Accepted)**: The notification request has been queued for sending.
- **Error Responses**:
    - `400 Bad Request`: Invalid data.

### 4.2. `GET /notify/preferences/{user_id}`
- **Description**: Retrieves the notification preferences for a given user.
- **Success Response (200 OK)**:
    ```json
    {
      "user_id": "...",
      "preferences": {
        "email": {
          "enabled": true,
          "address": "user@example.com"
        },
        "sms": {
          "enabled": false
        }
      }
    }
    ```
- **Error Responses**:
    - `404 Not Found`: User not found.

## 5. Workflow

1. A baggage scan event is published to the `scan_events` Kafka topic by the Baggage Tracking microservice.
2. The Notification microservice consumes the event.
3. It retrieves the user's notification preferences from the User Profile microservice.
4. It dispatches alerts via the user's preferred channels (e.g., SendGrid for email, Twilio for SMS).

## 6. Non-Functional Requirements

- **Configuration**: All external configuration, including API keys for SendGrid/Twilio/Firebase and Kafka broker addresses, must be loaded from **AWS Systems Manager Parameter Store** on startup.
- **Secrets Management**: All secrets, including API keys, must be securely fetched from **AWS Secrets Manager**.
- **API Documentation**: The service must generate a standard **OpenAPI 3 specification** from its code.
- **Logging**: The service must use **Structured Logging** (JSON format) and must propagate the `correlation-id` received in Kafka messages to all log messages.
- **Standardized Error Handling**: All API responses must adhere to the standardized error format defined in the Architecture Overview.
- **Service-Level Rate Limiting**: Critical internal endpoints or resource-intensive operations will implement their own rate limiting mechanisms.
- **Timeliness**: Notifications should be sent within 5 seconds of the event being consumed.
- **Reliability**: The service must guarantee at-least-once delivery of notifications.
- **Scalability**: The service must be able to handle a high volume of notifications during peak travel times.
