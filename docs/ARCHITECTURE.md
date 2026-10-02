# 🏗️ System Architecture & Design Document

This document describes the high-level system architecture, client-server interactions, data flow, and security mechanisms of the **Mini E-Commerce Demo Project**.

---

## 1. System Overview

The system is designed as a decoupled, stateless **MERN (MongoDB, Express.js, React.js, Node.js)** web application. Communication between the React Single Page Application (SPA) and the Express REST API server takes place exclusively over HTTP(S) with JSON payloads.

```mermaid
flowchart LR
    subgraph Client["Frontend Client (React + Vite)"]
        UI[Tailwind UI Components]
        Router[React Router v6]
        Context[Auth & Cart Contexts]
        Axios[Axios HTTP Client]
    end

    subgraph Server["Backend Server (Node.js + Express)"]
        Middlewares[Auth & Admin Middleware]
        Controllers[API Route Controllers]
        Validation[Request & Business Logic Validation]
        Mongoose[Mongoose ODM]
    end

    subgraph Database["MongoDB Database"]
        ColUser[(Users)]
        ColCat[(Categories)]
        ColProd[(Products)]
        ColOrder[(Orders)]
    end

    UI --> Context
    Context --> Router
    Router --> Axios
    Axios -- "HTTP REST Requests (Bearer JWT)" --> Middlewares
    Middlewares --> Controllers
    Controllers --> Validation
    Validation --> Mongoose
    Mongoose --> Database
```

---

## 2. Layered Architecture

### 2.1 Frontend Architecture (`/client`)

The client is built as a Single Page Application (SPA) using React.js and Vite:

1. **Routing Layer (`react-router-dom`)**:
   - **Public Routes**: Accessible by anyone (`/`, `/products`, `/products/:id`, `/login`, `/register`).
   - **Protected Customer Routes**: Requires an authenticated customer session (`/cart`, `/checkout`, `/my-orders`).
   - **Protected Admin Routes**: Requires an authenticated session with `role === 'admin'` (`/admin`, `/admin/categories`, `/admin/products`, `/admin/orders`).
2. **State Management**:
   - **AuthContext**: Manages user authentication state, token storage (`localStorage`), login, logout, and current user profile.
   - **CartContext**: Manages in-memory and persisted cart items, quantity limits based on available product stock, and subtotal calculations.
3. **Network Layer (Axios)**:
   - Configured with a `baseURL` (`import.meta.env.VITE_API_BASE_URL`).
   - Request Interceptor: Automatically attaches the JWT `Authorization: Bearer <token>` header if a token exists in `localStorage`.
   - Response Interceptor: Catches `401 Unauthorized` responses and triggers automatic logout if the token expires.
4. **Presentation Layer**:
   - Styled with utility-first **Tailwind CSS**.
   - Modular, reusable components (Modals, Tables, Cards, Badges, Loaders).

### 2.2 Backend Architecture (`/server`)

The backend follows the standard Express.js Controller-Model pattern:

1. **Middleware Pipeline**:
   - `cors`: Handles Cross-Origin Resource Sharing for the React client.
   - `express.json()`: Parses incoming JSON request bodies.
   - `authMiddleware`: Validates JWT tokens and attaches `req.user` to requests.
   - `adminMiddleware`: Verifies that `req.user.role === 'admin'`.
   - Global Error Handler: Catches unhandled exceptions and formats consistent JSON error responses.
2. **Controllers & Business Logic**:
   - Handles route handling, request validation, database interactions, and response formatting.
3. **Data Access Layer (Mongoose)**:
   - Schema enforcement, model validation, pre-save hooks (e.g., password hashing with bcrypt), and index definitions.

---

## 3. Authentication & Authorization Flow

Authentication uses stateless JSON Web Tokens (JWT).

```mermaid
sequenceDiagram
    autonumber
    actor Customer as Customer / Admin
    participant Client as React Client
    participant Server as Express Server
    participant DB as MongoDB

    Note over Customer,DB: Registration Flow
    Customer->>Client: Enters Name, Email, Password, Confirm Password
    Client->>Client: Validates passwords match & length >= 6
    Client->>Server: POST /api/auth/register
    Server->>DB: Check if Email exists
    alt Email already registered
        Server-->>Client: 400 Bad Request ("Email already in use")
    else Email available
        Server->>Server: Hash password with bcrypt (salt rounds = 10)
        Server->>DB: Save new User (role = 'customer')
        Server-->>Client: 201 Created (User data + JWT Token)
        Client->>Client: Save token to localStorage & update AuthContext
    end

    Note over Customer,DB: Login Flow
    Customer->>Client: Enters Email & Password
    Client->>Server: POST /api/auth/login
    Server->>DB: Find user by Email
    Server->>Server: bcrypt.compare(password, user.password)
    alt Invalid Credentials
        Server-->>Client: 401 Unauthorized ("Invalid email or password")
    else Valid Credentials
        Server->>Server: Generate JWT (userId, role, expires in 7d)
        Server-->>Client: 200 OK (Token + User Object)
        Client->>Client: Save token to localStorage & redirect based on role
    end
```

### Authorization Middleware Pipeline

```text
Incoming Request
      │
      ▼
authMiddleware
      │── 1. Check Authorization header: 'Bearer <token>'
      │── 2. Verify token signature with JWT_SECRET
      │── 3. Find user in MongoDB (excluding password)
      │── 4. Attach user object to req.user
      ▼
adminMiddleware (Only for Admin routes)
      │── 1. Inspect req.user.role
      │── 2. If req.user.role !== 'admin', return 403 Forbidden
      ▼
Route Controller Handler
```

---

## 4. Order Placement & Stock Integrity Logic

A critical business rule is ensuring **inventory integrity** and **price security**.

### 4.1 Security Principles
1. **Never trust frontend pricing**: The total calculation is performed exclusively on the server by fetching the current price of each item from the `Product` collection.
2. **Prevent race conditions & negative stock**: Ensure stock is checked and decremented atomically.
3. **Cash on Delivery (COD)**: Orders transition to `Pending` immediately upon creation.

### 4.2 Checkout Sequence

```mermaid
sequenceDiagram
    autonumber
    actor User as Customer
    participant Cart as React Cart State
    participant Checkout as Checkout Page
    participant Server as Order Controller
    participant DB as MongoDB

    User->>Checkout: Fills shipping details & clicks "Place Order"
    Checkout->>Server: POST /api/orders { products: [{ product: id, quantity }], shippingAddress }
    Server->>Server: Verify req.user from JWT

    loop For each item in order
        Server->>DB: Fetch Product by ID
        Server->>Server: Check if (product.stock < requested quantity)
        alt Insufficient Stock
            Server-->>Checkout: 400 Bad Request ("Product X is out of stock or requested quantity exceeds available stock")
        else Stock Available
            Server->>Server: Calculate item subtotal using DB price: (product.price * quantity)
        end
    end

    Server->>Server: Calculate totalAmount = sum(item subtotals)
    Server->>DB: Create Order document (status = 'Pending', totalAmount, products, shippingAddress)

    loop For each item in order
        Server->>DB: Update Product: decrement stock by quantity ($inc: { stock: -quantity })
    end

    Server-->>Checkout: 201 Created (Order Object)
    Checkout->>Cart: Clear cart items
    Checkout-->>User: Redirect to "My Orders" with success toast
```

---

## 5. Stock Constraint Enforcement

### In the Frontend (Client)
- On the **Product Details** page:
  - If `stock === 0`, disable the "Add to Cart" button and display "Out of Stock".
  - Quantity input is constrained between `1` and `product.stock`.
- In the **Shopping Cart**:
  - The `+` (increment) button is disabled when `item.quantity >= item.stock`.
  - An inline alert appears if a user attempts to add more than available stock.

### In the Backend (Server)
- When `POST /api/orders` executes:
  - Each product's stock is re-verified against MongoDB.
  - If any product cannot fulfill the quantity, the entire order is aborted with an informative error message.
  - Stock is decremented using MongoDB's `$inc` operator.

---

## 6. Error Handling Strategy

All API endpoints return predictable, standardized JSON error objects:

```json
{
  "success": false,
  "message": "Human readable error description",
  "error": "Detailed validation or system message (dev mode only)"
}
```

Standard HTTP status codes utilized:
- `200 OK`: Request succeeded.
- `201 Created`: Resource successfully created.
- `400 Bad Request`: Validation failure, missing fields, or insufficient stock.
- `401 Unauthorized`: Missing or invalid JWT token.
- `403 Forbidden`: Authenticated user lacks admin privileges.
- `404 Not Found`: Resource (product, category, or order) not found.
- `500 Internal Server Error`: Unexpected server exception.
