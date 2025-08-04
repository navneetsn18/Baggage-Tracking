
# PRD: Frontend (Next.js) (Hackathon Edition)

**Version**: 1.0 (Hackathon)
**Date**: 2025-08-04

## 1. Overview

This document outlines the product requirements for the Frontend of the Realtime Baggage Tracking System for a 2-day hackathon. The frontend will be a modern, responsive web application built with **Next.js** and the **shadcn/ui** component library to ensure a minimal, cool, and highly usable interface for a functional MVP.

## 2. UI/UX Philosophy

- **Design System**: The application will adhere to a minimalist design language, utilizing the **shadcn/ui** component library built on Tailwind CSS.
- **Theming**: A **Dark/Light mode** theme toggle is required for user comfort.
- **Components**: Standard shadcn/ui components will be used for consistency. This includes `Card` for displaying information, `Table` for data, `Dialog` for modals, `Input` for forms, and `Toast` for non-intrusive notifications.

## 3. Features

### 3.1. User Portal
- **Description**: A clean, intuitive portal for passengers to track their baggage.
- **Features**:
    - **Baggage Tracking**: Users can enter their PNR or baggage ID to view the current status and location history of their baggage, displayed in a `Card` component.
    - **Realtime Map**: A map (using Leaflet or Mapbox) that displays the flight route and the current location of the user's baggage. (Simplified: May use static map images or basic markers for hackathon).
    - **Notification History**: A basic view of notifications received about their baggage, presented in a `Table`.
- **Acceptance Criteria**:
    - The user portal must be fully responsive and provide a functional experience on both desktop and mobile devices.
    - Baggage tracking information should update reasonably quickly (near real-time for hackathon).

### 3.2. Admin Portal
- **Description**: A functional portal for airport staff to manage baggage, accessible only to authenticated `ADMIN` users.
- **Features**:
    - **Baggage Registration & QR Code Generation**:
        - A dedicated page with a form inside a `Card` for check-in staff to register new baggage.
        - The form will require a **PNR** and a passenger **Last Name**.
        - Upon submission, the interface will call the `POST /admin/register-baggage` endpoint.
        - On success, the page will display the newly generated **Baggage ID** and render a **printable QR code** (e.g., using a QR code library) containing that ID.
    - **QR Scanner**:
        - A mobile-responsive interface that uses the device's camera to scan baggage QR codes.
        - A successful scan will populate the baggage ID and trigger a call to the `POST /admin/scan` endpoint to update the baggage status.
    - **Reverse Baggage Lookup**:
        - A search bar for looking up baggage details.
        - Admins can enter a **Baggage ID** to query the `GET /admin/baggage/{baggage_id}` endpoint.
        - The interface will display the associated **PNR**, **Last Name**, and **Flight ID** in a `Card`.
    - **Flight-Centric View**:
        - A page allowing admins to search for a flight number.
        - The interface will display a list (`Table`) of all baggage associated with that flight, including their current status.
    - **Baggage Management Tools**:
        - Basic options to update baggage status or assign to a flight/carousel (simplified from previous reroute/lost features).
    - **Dashboard**:
        - A basic dashboard displaying key metrics like total scans or misplaced bags (simplified from previous detailed analytics).
- **Acceptance Criteria**:
    - The admin portal must be functional and accessible to authenticated admin users.
    - The baggage registration form must be usable and generate a scannable QR code.
    - The QR scanner must be functional.

### 3.3. Shared Features
- **Description**: Features common to both user and admin portals.
- **Features**:
    - **JWT-Authenticated Routes**: Routes will be protected by basic JWT authentication.
    - **Realtime Updates**: Basic real-time updates will be implemented (e.g., via simplified WebSockets or polling).
- **Acceptance Criteria**:
    - Authentication should work for both user and admin roles.
    - Basic real-time updates should be visible.

## 4. Non-Functional Requirements (Hackathon Focus)

- **User Experience**: The application must be intuitive and visually appealing, leveraging shadcn/ui.
- **Performance**: The application should load and respond quickly enough for a smooth hackathon demonstration.
- **Security**: Basic security measures (password hashing, JWTs) will be implemented.
- **Maintainability**: Code should be reasonably clean and understandable for a hackathon project.
