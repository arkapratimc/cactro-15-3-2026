# Cactro Event Management API (Full-Stack Demo)

A backend demonstration built with React Router v7 (Framework Mode). This project implements secure session management, Role-Based Access Control (RBAC), and asynchronous background processing.

**Live Production URL:** https://cactro-15-3-2026.vercel.app/

---

## Architectural Highlights

### 1. Identity and Access Management (IAM)
* **JWT-Powered Sessions:** Uses signed JSON Web Tokens stored in HttpOnly cookies for stateless, secure authentication.
* **Hybrid Auth Logic:** Support for both legacy default users (simple comparison) and new users (Bcrypt hashing), demonstrating a smooth migration path.
* **RBAC Guards:** Server-side validation ensures ORGANIZERS can only manage events and CUSTOMERS can only book them.

### 2. Async Job Queue (The Macro-task Pattern)
To keep the API responsive, the system offloads heavy operations to an in-memory job queue:
* **Non-Blocking:** Uses the Node.js Event Loop to offload operations like sending confirmation emails or broadcasting updates.
* **Decoupled Logic:** The API returns a success response to the user immediately, while the Worker processes the task on the next macro-task tick.

### 3. Data Integrity
* **Prefixed Auto-increment IDs:** Users and Events use semantic prefixes (e.g., mgr-1, cust-3, evt-101) for better log readability and debugging.
* **In-Memory Store:** A centralized db.server.ts acts as a single source of truth for the application state during the session.

---

## Rapid Testing Guide

I have included a pre-configured **testing-req.http** file in the root directory. This eliminates the need for manual Postman configuration.

### Requirements
* VS Code with the REST Client Extension installed.

### Instructions
1. Open testing-req.http.
2. Click "Send Request" on the endpoints in chronological order.
3. The extension handles the Set-Cookie headers automatically, maintaining the session across requests.

**Note on Logs:** If testing via the Live URL, background task outputs (Email confirmations and Notifications) are visible in the Vercel Project Logs. If running locally, they appear in the terminal 2 seconds after the request completes.

---

## Project Structure
* app/routes/auth/: Contains Login, Signup, Logout, and Session validation logic.
* app/routes/api/: Contains Event browsing, Detail management, and Ticket booking endpoints.
* app/utils/: Core utilities for JWT signing, Bcrypt hashing, and the Async Job Queue implementation.


Note on Local Builds

This project cannot be built locally using standard build commands as Vercel maintains their own proprietary template and build optimization process for React Router. To understand how the environment is structured and deployed, please refer to the Vercel Guide for more information.

## Route Configuration & Logic

The application uses a prefixed route structure for clear separation between authentication and data services.

### Authentication Routes (`/auth/*`)
| Route | Method | Logic |
| :--- | :--- | :--- |
| `auth/signup` | `POST` | Creates new user with **Bcrypt** password hashing. |
| `auth/login` | `POST` | Verifies credentials and issues a **JWT** via HttpOnly cookie. |
| `auth/logout` | `POST` | Invalidates the session by clearing the cookie. |
| `auth/me` | `GET` | Validates JWT to return current user data and role. |

### API Routes (`/api/*`)
| Route | Method | Logic |
| :--- | :--- | :--- |
| `api/events` | `GET` | Public browsing of all events. |
| `api/events/:id` | `GET` | Fetch specific event metadata. |
| `api/events/:id` | `PATCH` | **Organizer Only**: Updates title/description. Triggers async notification job. |
| `api/events/:id/book` | `POST` | **Customer Only**: Records booking. Triggers async email job. |