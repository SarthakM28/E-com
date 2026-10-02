# 📡 REST API Specification

This document provides the complete API reference for the **Mini E-Commerce Demo Project**.

---

## 1. General API Conventions

- **Base URL**: `http://localhost:5000/api`
- **Data Format**: `application/json` (both request body and responses)
- **Authentication**: JWT token sent via `Authorization: Bearer <token>` header for protected routes.

### Standard Success Response Format
```json
{
  "success": true,
  "data": { ... },
  "message": "Optional descriptive success message"
}
```

### Standard Error Response Format
```json
{
  "success": false,
  "message": "Human readable error description",
  "errors": [ ... ]
}
```

### Access Control Levels
| Label | Description |
| :--- | :--- |
| **Public** | Accessible by any client without authentication |
| **Customer** | Requires valid JWT token for any registered user |
| **Admin** | Requires valid JWT token with `role === 'admin'` |

---

## 2. API Summary Table

| Category | Method | Endpoint | Access | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Auth** | `POST` | `/api/auth/register` | Public | Register new customer account |
| **Auth** | `POST` | `/api/auth/login` | Public | Authenticate user (Customer or Admin) |
| **Auth** | `GET` | `/api/auth/me` | Customer | Fetch current authenticated user profile |
| **Categories** | `GET` | `/api/categories` | Public | List all product categories |
| **Categories** | `POST` | `/api/categories` | Admin | Create a new category |
| **Categories** | `PUT` | `/api/categories/:id` | Admin | Update an existing category |
| **Categories** | `DELETE`| `/api/categories/:id` | Admin | Delete a category |
| **Products** | `GET` | `/api/products` | Public | List & search products (category & search query) |
| **Products** | `GET` | `/api/products/:id` | Public | Get single product details by ID |
| **Products** | `POST` | `/api/products` | Admin | Create a new product |
| **Products** | `PUT` | `/api/products/:id` | Admin | Update product details or stock |
| **Products** | `DELETE`| `/api/products/:id` | Admin | Delete a product |
| **Orders** | `POST` | `/api/orders` | Customer | Create a new order with stock validation |
| **Orders** | `GET` | `/api/orders/my-orders`| Customer | List orders for the authenticated user |
| **Orders** | `GET` | `/api/admin/orders` | Admin | List all customer orders |
| **Orders** | `PATCH`| `/api/admin/orders/:id/status` | Admin | Update order fulfillment status |

---

## 3. Authentication Endpoints

### 3.1 Register Customer
Create a new customer account.

- **Method**: `POST`
- **Path**: `/api/auth/register`
- **Access**: Public

#### Request Body
```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "password123",
  "confirmPassword": "password123"
}
```

#### Validation Rules
- `name`: Required, trimmed.
- `email`: Required, valid email format, unique.
- `password`: Required, minimum 6 characters.
- `confirmPassword`: Required, must match `password`.

#### Response `201 Created`
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "user": {
      "_id": "660c1f5e8b4a2c001fa3d101",
      "name": "Jane Doe",
      "email": "jane@example.com",
      "role": "customer"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

#### Error Responses
- `400 Bad Request`: "Passwords do not match" or "Password must be at least 6 characters"
- `409 Conflict`: "Email is already registered"

---

### 3.2 Login User (Customer or Admin)
Authenticate an existing customer or administrator.

- **Method**: `POST`
- **Path**: `/api/auth/login`
- **Access**: Public

#### Request Body
```json
{
  "email": "jane@example.com",
  "password": "password123"
}
```

#### Response `200 OK`
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": {
      "_id": "660c1f5e8b4a2c001fa3d101",
      "name": "Jane Doe",
      "email": "jane@example.com",
      "role": "customer"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

#### Error Responses
- `401 Unauthorized`: "Invalid email or password"

---

### 3.3 Get Current Profile
Fetch the currently logged in user profile.

- **Method**: `GET`
- **Path**: `/api/auth/me`
- **Access**: Customer / Admin
- **Headers**: `Authorization: Bearer <token>`

#### Response `200 OK`
```json
{
  "success": true,
  "data": {
    "_id": "660c1f5e8b4a2c001fa3d101",
    "name": "Jane Doe",
    "email": "jane@example.com",
    "role": "customer"
  }
}
```

---

## 4. Category Endpoints

### 4.1 Get All Categories
Retrieve list of all categories.

- **Method**: `GET`
- **Path**: `/api/categories`
- **Access**: Public

#### Response `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "_id": "660c20108b4a2c001fa3d105",
      "name": "Electronics",
      "description": "Smartphones, laptops, and gadgets",
      "createdAt": "2026-10-02T10:05:00.000Z"
    },
    {
      "_id": "660c20108b4a2c001fa3d106",
      "name": "Fashion",
      "description": "Apparel, footwear, and accessories",
      "createdAt": "2026-10-02T10:06:00.000Z"
    }
  ]
}
```

---

### 4.2 Create Category
Create a new product category.

- **Method**: `POST`
- **Path**: `/api/categories`
- **Access**: Admin
- **Headers**: `Authorization: Bearer <admin-token>`

#### Request Body
```json
{
  "name": "Shoes",
  "description": "Running sneakers, casual shoes, and boots"
}
```

#### Response `201 Created`
```json
{
  "success": true,
  "message": "Category created successfully",
  "data": {
    "_id": "660c20108b4a2c001fa3d107",
    "name": "Shoes",
    "description": "Running sneakers, casual shoes, and boots",
    "createdAt": "2026-10-02T10:12:00.000Z"
  }
}
```

#### Error Responses
- `400 Bad Request`: "Category name is required"
- `409 Conflict`: "Category already exists"

---

### 4.3 Update Category
Edit an existing category.

- **Method**: `PUT`
- **Path**: `/api/categories/:id`
- **Access**: Admin
- **Headers**: `Authorization: Bearer <admin-token>`

#### Request Body
```json
{
  "name": "Footwear",
  "description": "Updated description for all footwear"
}
```

#### Response `200 OK`
```json
{
  "success": true,
  "message": "Category updated successfully",
  "data": {
    "_id": "660c20108b4a2c001fa3d107",
    "name": "Footwear",
    "description": "Updated description for all footwear",
    "updatedAt": "2026-10-02T10:15:00.000Z"
  }
}
```

---

### 4.4 Delete Category
Delete a category if no products are associated with it.

- **Method**: `DELETE`
- **Path**: `/api/categories/:id`
- **Access**: Admin
- **Headers**: `Authorization: Bearer <admin-token>`

#### Response `200 OK`
```json
{
  "success": true,
  "message": "Category deleted successfully"
}
```

#### Error Responses
- `400 Bad Request`: "Cannot delete category with associated products"
- `404 Not Found`: "Category not found"

---

## 5. Product Endpoints

### 5.1 List Products (with Search and Filter)
Fetch catalog products with optional filtering by category name/ID and keyword search.

- **Method**: `GET`
- **Path**: `/api/products`
- **Access**: Public
- **Query Parameters**:
  - `category` *(optional)*: Category ID or Category name (e.g. `electronics`)
  - `search` *(optional)*: Search query string matched against title/description (e.g. `phone`)

#### Example Request
```http
GET /api/products?category=electronics&search=phone HTTP/1.1
```

#### Response `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "_id": "660c21008b4a2c001fa3d110",
      "name": "Smartphone Pro Max",
      "description": "Latest 5G smartphone with 128GB storage",
      "price": 899.99,
      "image": "https://images.unsplash.com/photo-1511707171634-5f897ff02aa9?w=500",
      "category": {
        "_id": "660c20108b4a2c001fa3d105",
        "name": "Electronics"
      },
      "stock": 14,
      "createdAt": "2026-10-02T10:10:00.000Z"
    }
  ]
}
```

---

### 5.2 Get Product By ID
Retrieve full details for a single product.

- **Method**: `GET`
- **Path**: `/api/products/:id`
- **Access**: Public

#### Response `200 OK`
```json
{
  "success": true,
  "data": {
    "_id": "660c21008b4a2c001fa3d110",
    "name": "Smartphone Pro Max",
    "description": "Latest 5G smartphone with 128GB storage",
    "price": 899.99,
    "image": "https://images.unsplash.com/photo-1511707171634-5f897ff02aa9?w=500",
    "category": {
      "_id": "660c20108b4a2c001fa3d105",
      "name": "Electronics"
    },
    "stock": 14,
    "createdAt": "2026-10-02T10:10:00.000Z"
  }
}
```

#### Error Responses
- `404 Not Found`: "Product not found"

---

### 5.3 Create Product
Add a new product to the catalog.

- **Method**: `POST`
- **Path**: `/api/products`
- **Access**: Admin
- **Headers**: `Authorization: Bearer <admin-token>`

#### Request Body
```json
{
  "name": "Wireless Noise Cancelling Headphones",
  "description": "High fidelity over-ear Bluetooth headphones with active noise cancellation.",
  "price": 149.99,
  "image": "https://images.unsplash.com/photo-1505740420928-5e560c06d30e?w=500",
  "category": "660c20108b4a2c001fa3d105",
  "stock": 25
}
```

#### Validation Rules
- `name`: Required string.
- `price`: Required, number >= 0.
- `stock`: Required, integer >= 0.
- `category`: Required valid Category ObjectId.
- `image`: Required valid URL string.

#### Response `201 Created`
```json
{
  "success": true,
  "message": "Product created successfully",
  "data": {
    "_id": "660c21008b4a2c001fa3d110",
    "name": "Wireless Noise Cancelling Headphones",
    "description": "High fidelity over-ear Bluetooth headphones with active noise cancellation.",
    "price": 149.99,
    "image": "https://images.unsplash.com/photo-1505740420928-5e560c06d30e?w=500",
    "category": "660c20108b4a2c001fa3d105",
    "stock": 25,
    "createdAt": "2026-10-02T10:10:00.000Z"
  }
}
```

---

### 5.4 Update Product
Update an existing product's fields or adjust inventory stock.

- **Method**: `PUT`
- **Path**: `/api/products/:id`
- **Access**: Admin
- **Headers**: `Authorization: Bearer <admin-token>`

#### Request Body
```json
{
  "name": "Wireless Noise Cancelling Headphones - Gen 2",
  "price": 139.99,
  "stock": 30
}
```

#### Response `200 OK`
```json
{
  "success": true,
  "message": "Product updated successfully",
  "data": { ... }
}
```

---

### 5.5 Delete Product
Remove a product from the catalog.

- **Method**: `DELETE`
- **Path**: `/api/products/:id`
- **Access**: Admin
- **Headers**: `Authorization: Bearer <admin-token>`

#### Response `200 OK`
```json
{
  "success": true,
  "message": "Product deleted successfully"
}
```

---

## 6. Order Endpoints

### 6.1 Create Order (Checkout)
Place a new order with Cash on Delivery payment. The backend fetches current prices from MongoDB and updates stock.

- **Method**: `POST`
- **Path**: `/api/orders`
- **Access**: Customer
- **Headers**: `Authorization: Bearer <customer-token>`

#### Request Body
```json
{
  "products": [
    {
      "product": "660c21008b4a2c001fa3d110",
      "quantity": 2
    }
  ],
  "shippingAddress": {
    "name": "Jane Doe",
    "phone": "+1 555-0199",
    "address": "742 Evergreen Terrace",
    "city": "Springfield",
    "pincode": "97477"
  }
}
```

#### Server-side Logic
1. Validate `shippingAddress` fields (all required).
2. For each product ID, look up authoritative price and stock from the `Product` collection.
3. Reject if `quantity > product.stock`.
4. Calculate `totalAmount = sum(product.price * quantity)`.
5. Create `Order` record with `status: 'Pending'`.
6. Decrement `stock` in `Product` collection using `$inc: { stock: -quantity }`.

#### Response `201 Created`
```json
{
  "success": true,
  "message": "Order placed successfully",
  "data": {
    "_id": "660c22888b4a2c001fa3d150",
    "user": "660c1f5e8b4a2c001fa3d101",
    "products": [
      {
        "product": "660c21008b4a2c001fa3d110",
        "name": "Wireless Noise Cancelling Headphones",
        "price": 149.99,
        "quantity": 2
      }
    ],
    "totalAmount": 299.98,
    "shippingAddress": {
      "name": "Jane Doe",
      "phone": "+1 555-0199",
      "address": "742 Evergreen Terrace",
      "city": "Springfield",
      "pincode": "97477"
    },
    "status": "Pending",
    "createdAt": "2026-10-02T10:30:00.000Z"
  }
}
```

#### Error Responses
- `400 Bad Request`: "Product 'Wireless Noise Cancelling Headphones' does not have sufficient stock (Available: 1, Requested: 2)"

---

### 6.2 Get Customer's Orders
Retrieve all orders placed by the currently authenticated customer.

- **Method**: `GET`
- **Path**: `/api/orders/my-orders`
- **Access**: Customer
- **Headers**: `Authorization: Bearer <customer-token>`

#### Response `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "_id": "660c22888b4a2c001fa3d150",
      "products": [
        {
          "product": "660c21008b4a2c001fa3d110",
          "name": "Wireless Noise Cancelling Headphones",
          "price": 149.99,
          "quantity": 2
        }
      ],
      "totalAmount": 299.98,
      "shippingAddress": {
        "name": "Jane Doe",
        "phone": "+1 555-0199",
        "address": "742 Evergreen Terrace",
        "city": "Springfield",
        "pincode": "97477"
      },
      "status": "Pending",
      "createdAt": "2026-10-02T10:30:00.000Z"
    }
  ]
}
```

---

### 6.3 Get All Orders (Admin)
Retrieve all orders across all customers.

- **Method**: `GET`
- **Path**: `/api/admin/orders`
- **Access**: Admin
- **Headers**: `Authorization: Bearer <admin-token>`

#### Response `200 OK`
```json
{
  "success": true,
  "data": [
    {
      "_id": "660c22888b4a2c001fa3d150",
      "user": {
        "_id": "660c1f5e8b4a2c001fa3d101",
        "name": "Jane Doe",
        "email": "jane@example.com"
      },
      "products": [ ... ],
      "totalAmount": 299.98,
      "shippingAddress": { ... },
      "status": "Pending",
      "createdAt": "2026-10-02T10:30:00.000Z"
    }
  ]
}
```

---

### 6.4 Update Order Status (Admin)
Change the fulfillment status of an order.

- **Method**: `PATCH`
- **Path**: `/api/admin/orders/:id/status`
- **Access**: Admin
- **Headers**: `Authorization: Bearer <admin-token>`

#### Request Body
```json
{
  "status": "Shipped"
}
```

#### Allowed Status Values
- `Pending`
- `Confirmed`
- `Shipped`
- `Delivered`
- `Cancelled`

#### Response `200 OK`
```json
{
  "success": true,
  "message": "Order status updated successfully",
  "data": {
    "_id": "660c22888b4a2c001fa3d150",
    "status": "Shipped",
    "updatedAt": "2026-10-02T11:00:00.000Z"
  }
}
```

#### Error Responses
- `400 Bad Request`: "Invalid order status value"
- `404 Not Found`: "Order not found"
