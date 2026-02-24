# Community API & Hub

A full-stack web application built with a Node.js/Express backend and a Vanilla JS/HTML/CSS frontend. It serves as a community platform where users can register, login, create posts, and read posts from others. It implements robust authentication, role-based authorization, and a clean, paginated REST API.

---

## 🚀 Tech Stack

### Backend
*   **Runtime:** Node.js
*   **Framework:** Express.js
*   **Database:** MongoDB
*   **ODM:** Mongoose
*   **Authentication:** JSON Web Tokens (JWT) & bcryptjs (password hashing)
*   **Environment Management:** dotenv

### Frontend
*   **Structure:** HTML5
*   **Styling:** Vanilla CSS (Modern UI with CSS Variables, Flexbox/Grid, Glassmorphism)
*   **Logic:** Vanilla JavaScript (Fetch API for HTTP requests, LocalStorage for state)

---

## ✨ Features Implemented

### 1. User Authentication & Authorization (JWT)
*   User registration with hashed passwords.
*   User login returning a secure JWT token.
*   Middleware to protect routes (`protect`), ensuring only authenticated users can access certain endpoints.
*   Frontend securely stores the token in `localStorage` and attaches it as a `Bearer` token in the `Authorization` header for protected requests.

### 2. Role-Based Access Control (RBAC)
*   Users have a `role` field in the database (`user` or `admin`).
*   **Normal Users:** Can create posts and edit/delete **only their own** posts.
*   **Admins:** Have elevated privileges via the `admin` middleware, allowing them to delete **any** post regardless of ownership.

### 3. Content CRUD (RESTful API)
*   **Create:** Authenticated users can create new posts (Title & Content).
*   **Read:** Anyone (Public) can view the feed of posts. Posts are populated with the author's name and email using Mongoose's `.populate()`.
*   **Delete:** Enforces strict ownership or admin-level checks before allowing deletion.

### 4. Advanced API design
*   **Pagination:** The `GET /api/posts` endpoint supports `page` and `limit` query parameters, ensuring scalable performance instead of loading the entire database at once.
*   **Global Error Handling:** Instead of repetitive `try/catch` and `res.status(500)` blocks in every controller, a centralized `error.middleware.js` intercepts all errors, returning a standard, formatted JSON error response (including stack traces in development).

### 5. Dynamic Vanilla Frontend
*   A responsive, dark-themed UI served statically via Express (`public` folder).
*   Dynamic DOM manipulation to show/hide the "Delete" button only if the logged-in user is the author or an admin.
*   Modal interface for seamless Login/Registration switching.
*   State management tracking `currentUser` and `currentToken`.

---

## 📂 Project Structure

```text
community-api/
│
├── public/                 # Static Frontend Files
│   ├── index.html          # Main HTML structure & UI templates
│   ├── style.css           # Modern CSS styling (Flex/Grid/CSS Variables)
│   └── app.js              # Frontend logic (Auth, Fetch API, DOM manipulation)
│
├── src/                    # Backend Source Code
│   ├── config/
│   │   └── db.js           # Mongoose MongoDB connection setup
│   │
│   ├── controllers/        # Request/Response Handling Logic
│   │   ├── auth.controller.js # registerUser, loginUser
│   │   └── post.controller.js # getPosts (paginated), createPost, deletePost
│   │
│   ├── middleware/         # Custom Express Middleware
│   │   ├── auth.middleware.js # `protect` (verifies JWT) & `admin` (checks role)
│   │   └── error.middleware.js# Global error handler for clean JSON responses
│   │
│   ├── models/             # Mongoose Database Schemas
│   │   ├── User.js         # User Schema (name, email, password, role)
│   │   └── Post.js         # Post Schema (title, content, author reference)
│   │
│   ├── routes/             # API Route Definitions
│   │   ├── auth.routes.js  # /api/auth/register, /api/auth/login
│   │   └── post.routes.js  # /api/posts (GET, POST, DELETE)
│   │
│   └── server.js           # Express App Entry Point (Middleware, Routing, Static Server)
│
├── .env                    # Environment variables (MONGO_URI, PORT, JWT_SECRET)
└── package.json            # Project dependencies & scripts
```

---

## ⚙️ How It Works (The Flow)

1.  **Frontend Request:** The user fills out the "Create Post" form and clicks Submit. `app.js` runs `handleCreatePost()`.
2.  **Fetch API:** The frontend uses `fetch()` to send a `POST` request to `/api/posts`, attaching the JWT token (stored during login) in the `Authorization` header.
3.  **Express Server:** The request hits `server.js` and is routed to `post.routes.js`.
4.  **Auth Middleware:** Before the request hits the controller, the `protect` middleware intercepts it. It extracts the JWT from the header, verifies it with `JWT_SECRET`, finds the user in MongoDB, and attaches `req.user`. If the token is invalid, it throws a 401 error.
5.  **Controller Logic:** The request proceeds to `createPost` in `post.controller.js`. It grabs the `title` and `content` from `req.body` and creates a new document in the `Post` model, automatically setting the `author` to `req.user.id`.
6.  **Response:** The controller sends a `201 Created` JSON response back to the frontend.
7.  **Frontend Update:** `app.js` receives the success response, immediately calls `fetchPosts(1)` to grab the newest paginated list of posts, and updates the DOM to show the new post at the top of the feed.
