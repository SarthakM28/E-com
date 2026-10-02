# 🎨 Frontend Specification & UI/UX Design

This document details the frontend architecture, page layouts, component hierarchy, state management, and design system for the **Mini E-Commerce Demo Project**.

The frontend is built with **React.js (Vite)** and styled using **Tailwind CSS**.

---

## 1. Design Principles & UI Guidelines

1. **Clean & Minimalist**: Uncluttered whitespace, clear typographic hierarchy, subtle borders, and smooth hover transitions.
2. **Mobile-First & Fully Responsive**: Works across mobile phones (`<640px`), tablets (`640px - 1023px`), and desktops (`>=1024px`).
3. **Intuitive Feedback**:
   - Dynamic toasts for actions (added to cart, order placed, error messages).
   - Loading skeletons and spinners during API fetching.
   - Informative empty states when categories, products, or orders are empty.
   - Confirmation dialogs before destructive actions (e.g., deleting a product or category).
4. **No Over-Engineering**: Clean vanilla Tailwind utility classes without bloated third-party UI suites.

### Color Palette (Tailwind Tokens)

| Element | Tailwind Class | Hex / Role |
| :--- | :--- | :--- |
| **Primary Brand** | `bg-indigo-600`, `hover:bg-indigo-700` | CTA buttons, active tabs, brand accents |
| **Background** | `bg-gray-50` / `bg-white` | Body background and card surfaces |
| **Text Primary** | `text-gray-900` | Headings, titles, prices |
| **Text Muted** | `text-gray-500` | Descriptions, timestamps, meta info |
| **Borders** | `border-gray-200` | Divider lines, card outlines, table borders |
| **Status Pending** | `bg-amber-100 text-amber-800` | Order status badge: Pending |
| **Status Confirmed**| `bg-blue-100 text-blue-800` | Order status badge: Confirmed |
| **Status Shipped** | `bg-indigo-100 text-indigo-800`| Order status badge: Shipped |
| **Status Delivered**| `bg-emerald-100 text-emerald-800`| Order status badge: Delivered |
| **Status Cancelled**| `bg-rose-100 text-rose-800` | Order status badge: Cancelled |

---

## 2. Route Architecture

Configured using `react-router-dom`:

```text
/ (Public Layout)
├── / ........................... Home Page
├── /products .................. Product Catalog (Search & Category filter)
├── /products/:id .............. Product Details
├── /login ..................... Customer & Admin Login
├── /register .................. Customer Registration
│
├── (Protected Customer Routes - requires Login)
│   ├── /cart .................. Shopping Cart
│   ├── /checkout .............. Checkout & Shipping Address
│   └── /my-orders ............. Customer Order History
│
└── /admin (Protected Admin Layout - requires Admin role)
    ├── /admin ................. Admin Dashboard Overview
    ├── /admin/categories ...... Category Management (CRUD)
    ├── /admin/products ........ Product Management (CRUD & Stock)
    └── /admin/orders .......... Order Management & Status Updates
```

---

## 3. Global State Management

### 3.1 `AuthContext`
Provides global authentication state throughout the app:
- `user`: Current user object (`_id`, `name`, `email`, `role`).
- `token`: JWT string stored in `localStorage`.
- `isAuthenticated`: Boolean flag.
- `isAdmin`: Computed boolean (`user?.role === 'admin'`).
- `login(email, password)`: Authenticates, saves token, sets user.
- `register(userData)`: Registers, saves token, sets user.
- `logout()`: Clears token from `localStorage` and resets state.

### 3.2 `CartContext`
Provides persistent shopping cart state:
- `cartItems`: Array of `{ product, quantity }`.
- `addToCart(product, quantity = 1)`:
  - Checks if item already exists in cart.
  - Ensures `(existingQty + quantity) <= product.stock`.
  - Displays error toast if requested quantity exceeds stock.
- `updateQuantity(productId, quantity)`:
  - Constrained strictly between `1` and `product.stock`.
- `removeFromCart(productId)`: Removes item.
- `clearCart()`: Empties cart upon successful checkout.
- `totalItems`: Total unit count for Navbar badge.
- `totalPrice`: Computed subtotal of all items.

---

## 4. Pages & UI Specifications

### 4.1 Navigation Bar (`components/Navbar.jsx`)
- **Desktop**:
  - Logo (`ShopMini`).
  - Navigation links: `Home`, `Products`.
  - Search bar input with immediate search or enter trigger.
  - Cart button with badge showing `totalItems`.
  - User profile menu:
    - If Guest: "Login" & "Register" buttons.
    - If Customer: "My Orders" link and "Logout" button.
    - If Admin: "Admin Panel" shortcut button and "Logout".
- **Mobile**:
  - Hamburger menu toggling a slide-out drawer containing all nav links.

---

### 4.2 Home Page (`pages/Home.jsx`)
- **Hero Banner**: Eye-catching callout promoting featured collections with "Shop Now" button directing to `/products`.
- **Category Quick-Filter**: Visual pills/cards for quick navigation to popular categories.
- **Featured Products**: Responsive 4-column grid of top in-stock products.
- **Value Propositions**: 3-card banner (Free Fast Delivery, Cash on Delivery, 100% Quality Guaranteed).

---

### 4.3 Products Catalog Page (`pages/Products.jsx`)
- **Category Filter Bar**:
  - Horizontal pill list: `All | Electronics | Fashion | Shoes | ...`
  - Clicking a pill highlights it and updates query params (`?category=fashion`).
- **Search Bar**:
  - Input field to filter products in real-time or on submit (`?search=query`).
- **Product Grid**:
  - Responsive grid: 1 column on mobile, 2 on tablet, 3-4 on desktop (`grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-6`).
  - **Product Card (`components/ProductCard.jsx`)**:
    - Image with hover zoom effect.
    - Category badge (e.g., `Electronics`).
    - Title with line-clamp.
    - Price formatted in currency ($XX.XX).
    - Stock status: In Stock badge or "Out of Stock" (grayed out).
    - "Add to Cart" button (disabled if `stock === 0`).
- **Empty State**: Friendly illustration and message when no products match search or category.

---

### 4.4 Product Details Page (`pages/ProductDetails.jsx`)
- 2-column layout on desktop, stacked on mobile:
  - **Left**: Large high-resolution product image.
  - **Right**:
    - Category badge.
    - Product title.
    - Formatted price.
    - Full product description.
    - Stock indicator:
      - `In Stock (X available)` in green.
      - `Out of Stock` in red if `stock === 0`.
    - Quantity selector (`-` and `+` buttons) capped at `product.stock`.
    - "Add to Cart" button with cart icon.
    - Breadcrumbs for easy navigation back to `/products`.

---

### 4.5 Shopping Cart Page (`pages/Cart.jsx`)
- **Cart Table / List**:
  - Item thumbnail, product name, and category.
  - Unit price.
  - Quantity controls (`-` and `+`) with live stock limit checking:
    - Disable `+` when `item.quantity === product.stock`.
  - Item subtotal.
  - Remove item button with trash icon.
- **Order Summary Sidebar**:
  - Subtotal.
  - Shipping fee (Free).
  - Estimated Total.
  - "Proceed to Checkout" primary CTA button.
- **Empty Cart State**: Shows shopping bag icon with "Your cart is empty" and "Continue Shopping" button.

---

### 4.6 Checkout Page (`pages/Checkout.jsx`)
- **Shipping Address Form**:
  - Full Name (required)
  - Phone Number (required)
  - Street Address (required)
  - City (required)
  - Pincode / Postal Code (required)
- **Payment Method Section**:
  - Pre-selected radio card: **Cash on Delivery (COD)**.
  - Information note: *"Pay with cash upon package arrival."*
- **Order Review Summary**:
  - Mini list of ordered items, quantities, and prices.
  - Total amount due.
  - "Place Order" button with loading spinner state during submission.

---

### 4.7 My Orders Page (`pages/MyOrders.jsx`)
- Displays all historical orders placed by the customer.
- Each order card displays:
  - Order ID & placement date.
  - Status badge with color coding (`Pending`, `Confirmed`, `Shipped`, `Delivered`, `Cancelled`).
  - Itemized product list with thumbnails, names, and checkout prices.
  - Delivery address summary.
  - Total order amount.
- Empty state if user has placed no orders yet.

---

### 4.8 Authentication Pages (`Login.jsx` & `Register.jsx`)
- Centered modern cards with clean shadow and border.
- **Register**:
  - Fields: Name, Email, Password, Confirm Password.
  - Form validation: Valid email format, password matching, minimum 6 characters.
- **Login**:
  - Fields: Email, Password.
  - Error alert banner on invalid credentials.
  - Redirects admin users automatically to `/admin` and customers to `/products` or `/cart`.

---

## 5. Admin Panel Specifications

Admin routes use an **Admin Layout** with a responsive persistent sidebar on desktop and toggleable drawer on mobile.

```
┌──────────────────────────────────────────────────────────┐
│ Header: Admin Dashboard       [Store Front Link] [Logout]│
├──────────────┬───────────────────────────────────────────┤
│ Sidebar:     │ Content Area:                             │
│ • Dashboard  │                                           │
│ • Categories │ [Table of Products / Categories / Orders] │
│ • Products   │ [+ Add New Button]                        │
│ • Orders     │                                           │
└──────────────┴───────────────────────────────────────────┘
```

### 5.1 Admin Categories (`pages/admin/AdminCategories.jsx`)
- "Add Category" button opening modal.
- Responsive table listing:
  - Category Name
  - Description
  - Actions: Edit (modal) and Delete.
- **Delete Confirmation Dialog**: Prevents accidental deletion; backend blocks deletion if products are linked.

### 5.2 Admin Products (`pages/admin/AdminProducts.jsx`)
- "Add Product" button opening form modal.
- Responsive table listing:
  - Thumbnail image
  - Product Name
  - Category name
  - Price ($)
  - Current Stock count (highlighted in red if `stock <= 5`)
  - Actions: Edit (modal) and Delete (confirmation dialog).
- **Add / Edit Product Modal**:
  - Fields: Product Name, Description, Price, Image URL, Category (dropdown), Stock.

### 5.3 Admin Orders (`pages/admin/AdminOrders.jsx`)
- Comprehensive table listing all customer orders:
  - Order ID & Date
  - Customer Name & Email
  - Items summary (e.g. `2x Headphones, 1x Shoes`)
  - Total Amount ($)
  - Status Dropdown Selector:
    - Allows immediate update (`Pending`, `Confirmed`, `Shipped`, `Delivered`, `Cancelled`).
    - Updating triggers `PATCH /api/admin/orders/:id/status` with toast notification.
  - Order details modal showing full shipping address and contact phone.

---

## 6. Feedback & Interactive Elements

- **Toast Notifications**: Reusable toast component (or `react-hot-toast`) for feedback:
  - Success: Item added to cart, order placed, category updated.
  - Error: Stock exceeded, invalid credentials, deletion blocked.
- **Confirmation Modals**: Standardized modal for destructive actions:
  - Title: *"Delete Product?"*
  - Body: *"Are you sure you want to delete this product? This action cannot be undone."*
  - Action buttons: *"Cancel"* and *"Delete"*.
- **Loading Skeletons**: Card and table skeletons displayed while awaiting network responses to prevent jarring layout shifts.
