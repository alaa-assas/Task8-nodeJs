Task Management System API Design 
Project Structure 


## Project Structure
```bash
task-management-system/
├── src/
│   ├── controllers/          
│   │   ├── auth.controller.js
│   │   ├── user.controller.js
│   │   ├── project.controller.js
│   │   ├── task.controller.js
│   │   └── notification.controller.js
│   ├── middleware/            
│   │   ├── auth.middleware.js
│   │   ├── error.middleware.js
│   │   └── notFound.middleware.js
│   ├── models/                
│   │   ├── User.js
│   │   ├── Project.js
│   │   ├── Task.js
│   │   └── Notification.js
│   ├── routes/                
│   │   ├── auth.routes.js
│   │   ├── user.routes.js
│   │   ├── project.routes.js
│   │   ├── task.routes.js
│   │   └── notification.routes.js
│   ├── services/              
│   │   ├── email.service.js
│   │   └── notification.service.js
│   ├── utils/                 
│   │   ├── passwordService.js
│   │   ├── cookieService.js
│   │   └── tokenService.js
│   └── app.js                 
├── .env                       
├── .gitignore
├── package.json
└── README.md
 ```
 
 ## Required Libraries

### Core Dependencies

**express**  
- **Purpose**: Web framework for building RESTful APIs  
- **Why needed**: Handles routing, middleware, and HTTP server setup to create a structured and scalable backend

**mongoose**  
- **Purpose**: Object Data Modeling (ODM) library for MongoDB  
- **Why needed**: Enables structured data modeling, schema validation, and relationship management between collections

**jsonwebtoken**  
- **Purpose**: Implements JSON Web Token (JWT) authentication  
- **Why needed**: Secures protected routes with stateless, token-based user verification without server-side sessions

**bcryptjs**  
- **Purpose**: Password hashing and comparison library  
- **Why needed**: Ensures user passwords are securely hashed with salt before storage, preventing exposure even if the database is compromised

**dotenv**  
- **Purpose**: Loads environment variables from a `.env` file into `process.env`  
- **Why needed**: Keeps sensitive configuration (e.g., database URLs, JWT secrets) out of source code for better security and environment management

**express-validator**  
- **Purpose**: Input validation and sanitization middleware for Express  
- **Why needed**: Validates and cleans user input (e.g., emails, passwords) to prevent invalid data and security risks like injection attacks

**cors**  
- **Purpose**: Middleware to enable Cross-Origin Resource Sharing  
- **Why needed**: Allows frontend applications on different origins (domains/ports) to safely communicate with the backend API

**helmet**  
- **Purpose**: Security middleware that sets safe HTTP headers  
- **Why needed**: Mitigates common web vulnerabilities such as XSS, clickjacking, and MIME sniffing by hardening response headers

**morgan**  
- **Purpose**: HTTP request logger middleware  
- **Why needed**: Logs incoming requests (method, URL, status, response time) to support debugging, monitoring, and auditing

**node-cron**  
- **Purpose**: Task scheduler for Node.js applications  
- **Why needed**: Automates periodic background jobs like sending task reminders, checking deadlines, and generating daily analytics


## Database Models (Mongoose Schemas)

### User Model
```javascript
{
  name: {
    type: String,
    required: [true, "Name is required"],
    trim: true
  },
  email: {
    type: String,
    required: [true, "Email is required"],
    unique: true,
    lowercase: true,
    match: [/^\S+@\S+\.\S+$/, "Please enter a valid email"]
  },
  password: {
    type: String,
    required: [true, "Password is required"],
    minlength: [8, "Password must be at least 8 characters"],
    select: false
  },
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  },
  createdAt: {
    type: Date,
    default: Date.now
  },
  updatedAt: {
    type: Date,
    default: Date.now
  }
}
```
---

### Project Model
```javascript
{
  name: {
    type: String,
    required: [true, "Project name is required"],
    trim: true,
    maxlength: 100
  },
  description: {
    type: String,
    trim: true,
    maxlength: 500
  },
  owner: {
    type: mongoose.Schema.ObjectId,
    ref: 'User',
    required: true
  },
  members: [{
    user: {
      type: mongoose.Schema.ObjectId,
      ref: 'User',
      required: true
    },
    role: {
      type: String,
      enum: ['member', 'admin'],
      default: 'member'
    },
    joinedAt: {
      type: Date,
      default: Date.now
    }
  }],
  createdAt: {
    type: Date,
    default: Date.now
  },
  updatedAt: {
    type: Date,
    default: Date.now
  }
}
```
**Relationships**:  
- `owner`: References a single **User** (the creator)  
- `members`: Array of references to **User** documents, representing collaborators

---

### Task Model
```javascript
{
  title: {
    type: String,
    required: [true, "Task title is required"],
    trim: true,
    maxlength: 150
  },
  description: {
    type: String,
    trim: true,
    maxlength: 1000
  },
  status: {
    type: String,
    enum: ['todo', 'in progress', 'done'],
    default: 'todo'
  },
  priority: {
    type: String,
    enum: ['low', 'medium', 'high'],
    default: 'medium'
  },
  dueDate: {
    type: Date,
    validate: {
      validator: function (value) {
        return !value || value > Date.now();
      },
      message: "Due date must be in the future"
    }
  },
  reminder: {
    type: Date
  },
  project: {
    type: mongoose.Schema.ObjectId,
    ref: 'Project',
    required: true
  },
  assignedTo: {
    type: mongoose.Schema.ObjectId,
    ref: 'User',
    default: null
  },
  createdBy: {
    type: mongoose.Schema.ObjectId,
    ref: 'User',
    required: true
  },
  completedAt: {
    type: Date,
    default: null
  },
  createdAt: {
    type: Date,
    default: Date.now
  },
  updatedAt: {
    type: Date,
    default: Date.now
  }
}
```
**Relationships**:  
- `project`: References a single **Project**  
- `assignedTo`: Optional reference to a **User** (the assignee)  
- `createdBy`: References the **User** who created the task

---

### Notification Model
```javascript
{
  user: {
    type: mongoose.Schema.ObjectId,
    ref: 'User',
    required: true
  },
  title: {
    type: String,
    required: [true, "Notification title is required"],
    trim: true
  },
  message: {
    type: String,
    required: [true, "Notification message is required"],
    trim: true
  },
  type: {
    type: String,
    enum: ['assignment', 'reminder', 'deadline', 'invitation'],
    required: true
  },
  relatedTask: {
    type: mongoose.Schema.ObjectId,
    ref: 'Task'
  },
  relatedProject: {
    type: mongoose.Schema.ObjectId,
    ref: 'Project'
  },
  isRead: {
    type: Boolean,
    default: false
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
}
```
**Relationships**:  
- `user`: References the **User** who receives the notification  
- `relatedTask` (optional): Links to a specific **Task**  
- `relatedProject` (optional): Links to a specific **Project**


## API Endpoints

## POST /api/auth/register  
- **Method**: POST  
- **Description**: Create a new user account  
- **Authentication**: Not required  
- **Request Body**:   
  - name (string, required)  
  - email (string, required, must be unique)  
  - password (string, required, minimum 6 characters)  
- **Response Success (201)**:   
  `{ success: true, message: "User registered successfully", data: { user: { id, name, email, role, createdAt }, token: "..." } }`  
- **Response Errors**:  
  - 400: `{ success: false, error: "Email already exists" }`  
  - 400: `{ success: false, error: "Password must be at least 8 characters" }`  
  - 400: `{ success: false, error: "All fields are required" }`  
  - 400: `{ success: false, error: "Invalid email format" }`  
- **Potential User Mistakes**:  
  - Submitting without filling all required fields  
  - Using an email that already exists  
  - Entering a weak or short password  

---

## POST /api/auth/login  
- **Method**: POST  
- **Description**: Authenticate user and return JWT token  
- **Authentication**: Not required  
- **Request Body**:   
  - email (string, required)  
  - password (string, required)  
- **Response Success (200)**:   
  `{ success: true, message: "Login successful", data: { user: { id, name, email, role }, token: "..." } }`  
- **Response Errors**:  
  - 400: `{ success: false, error: "Invalid credentials" }`  
  - 400: `{ success: false, error: "All fields are required" }`  
- **Potential User Mistakes**:  
  - Entering incorrect email or password  
  - Leaving one or both fields empty  

---

## GET /api/auth/me  
- **Method**: GET  
- **Description**: Get current authenticated user profile  
- **Authentication**: Required (JWT token in Authorization header)  
- **Request Body**: None  
- **Response Success (200)**:   
  `{ success: true, data: { id, name, email, role, avatar, createdAt } }`  
- **Response Errors**:  
  - 401: `{ success: false, error: "Authentication required" }`  
  - 404: `{ success: false, error: "User not found" }`  
- **Potential User Mistakes**:  
  - Missing or invalid JWT token  

---

## POST /api/projects  
- **Method**: POST  
- **Description**: Create a new project  
- **Authentication**:  Required (JWT token in header)  
- **Request Body**:   
  - name (string, required, max 100 characters)  
  - description (string, optional, max 500 characters)  
- **Response Success (201)**:   
  `{ success: true, data: { id, name, description, owner, members, createdAt } }`  
- **Response Errors**:  
  - 401: `{ success: false, error: "Authentication required" }`  
  - 400: `{ success: false, error: "Project name is required" }`  
  - 400: `{ success: false, error: "Project name cannot exceed 100 characters" }`  
- **Potential User Mistakes**:  
  - Not providing a project name  
  - Providing an excessively long name or description  

---

## GET /api/projects  
- **Method**: GET  
- **Description**: Get all projects the user is a member of  
- **Authentication**: Required  
- **Query Parameters**: None  
- **Response Success (200)**:   
  `{ success: true, data: [ { id, name, description, owner, members, createdAt }, ... ] }`  
- **Response Errors**:  
  - 401: `{ success: false, error: "Authentication required" }`  
- **Potential User Mistakes**:  
  - Missing authentication token  

---

## POST /api/projects/:projectId/invite  
- **Method**: POST  
- **Description**: Invite team members to a project by email  
- **Authentication**: Required (user must be project owner or admin)  
- **Request Body**:   
  - emails (array of strings, required, each must be valid email)  
- **Response Success (200)**:   
  `{ success: true, message: "Invitations sent successfully", data: { invited: [...], failed: [...] } }`  
- **Response Errors**:  
  - 401: `{ success: false, error: "Authentication required" }`  
  - 403: `{ success: false, error: "You are not authorized to invite members to this project" }`  
  - 404: `{ success: false, error: "Project not found" }`  
  - 400: `{ success: false, error: "No valid emails provided" }`  
- **Potential User Mistakes**:  
  - Trying to invite to a project they don’t own  
  - Sending invalid or non-existent emails  

---

## POST /api/tasks  
- **Method**: POST  
- **Description**: Create a new task inside a project  
- **Authentication**: Required (user must be a project member)  
- **Request Body**:   
  - title (string, required)  
  - description (string, optional)  
  - status (string, enum: 'todo', 'in progress', 'done', default: 'todo')  
  - priority (string, enum: 'low', 'medium', 'high', default: 'medium')  
  - dueDate   
  - reminder   
  - project (string, ObjectId, required)  
  - assignedTo (string, ObjectId, optional)  
- **Response Success (201)**:   
  `{ success: true, data: { id, title, description, status, priority, dueDate, reminder, project, assignedTo, createdBy, createdAt } }`  
- **Response Errors**:  
  - 401: `{ success: false, error: "Authentication required" }`  
  - 403: `{ success: false, error: "You are not a member of this project" }`  
  - 404: `{ success: false, error: "Project not found" }`  
  - 400: `{ success: false, error: "Title is required" }`  
  - 400: `{ success: false, error: "Due date must be in the future" }`  
- **Potential User Mistakes**:  
  - Assigning task to a user not in the project  
  - Setting a past due date  
  - Omitting the project ID  

---

## GET /api/tasks  
- **Method**: GET  
- **Description**: Get all tasks for the logged-in user (created or assigned)  
- **Authentication**: Required  
- **Query Parameters**:  
  - status (string: 'todo', 'in progress', 'done')  
  - project (string: valid project ID)  
  - search (string: case-insensitive match in title)  
  - page (number, default: 1)  
  - limit (number, default: 10, max: 50)  
- **Response Success (200)**:   
  `{ success: true, data: { tasks: [...], pagination: { page, pages, limit, total } } }`  
- **Response Errors**:  
  - 401: `{ success: false, error: "Authentication required" }`  
  - 400: `{ success: false, error: "Invalid project ID" }`  
- **Potential User Mistakes**:  
  - Forgetting to include authentication token  
  - Using an invalid status value  
  - Passing a non-numeric page or limit  

---

## GET /api/tasks/me  
- **Method**: GET  
- **Description**: Get all tasks assigned to the logged-in user  
- **Authentication**: Required  
- **Query Parameters**:  
  - status (string: 'todo', 'in progress', 'done')  
  - project (string: project ID)  
- **Response Success (200)**:   
  `{ success: true, data: [ { id, title, status, priority, dueDate, project, ... }, ... ] }`  
- **Response Errors**:  
  - 401: `{ success: false, error: "Authentication required" }`  
- **Potential User Mistakes**:  
  - Missing JWT token  

---

## PUT /api/tasks/:taskId  
- **Method**: PUT  
- **Description**: Update an existing task  
- **Authentication**: Required (user must own the task or be a project member)  
- **Request Body**: (all fields optional)  
  - title (string)  
  - description (string)  
  - status (string, enum: 'todo', 'in progress', 'done')  
  - priority (string, enum: 'low', 'medium', 'high')  
  - dueDate (ISO string)  
  - reminder (ISO string)  
  - assignedTo (string, ObjectId)  
- **Response Success (200)**:   
  `{ success: true, data: { updated task object } }`  
- **Response Errors**:  
  - 401: `{ success: false, error: "Authentication required" }`  
  - 403: `{ success: false, error: "You are not authorized to update this task" }`  
  - 404: `{ success: false, error: "Task not found" }`  
  - 400: `{ success: false, error: "Cannot assign task to user not in project" }`  
- **Potential User Mistakes**:  
  - Trying to update a task from a project they’re not part of  
  - Assigning to a non-member  

---

## GET /api/analytics/tasks  
- **Method**: GET  
- **Description**: Get task completion statistics for the current user  
- **Authentication**: Required  
- **Query Parameters**:  
  - period (string: 'week', 'month', 'year', default: 'month')  
- **Response Success (200)**:   
  ```json
  {
    "success": true,
    "data": {
      "totalTasks": 24,
      "completedTasks": 18,
      "completionRate": 75,
      "tasksByStatus": { "todo": 3, "in progress": 3, "done": 18 },
      "tasksByPriority": { "low": 5, "medium": 12, "high": 7 },
      "completionTimeline": [
        { "date": "2025-11-17", "completed": 2 },
        { "date": "2025-11-18", "completed": 3 }
      ]
    }
  }
  ```  
- **Response Errors**:  
  - 401: `{ success: false, error: "Authentication required" }`  
  - 400: `{ success: false, error: "Invalid period value" }`  
- **Potential User Mistakes**:  
  - Using unsupported period value (e.g., "day")  

---

## GET /api/notifications  
- **Method**: GET  
- **Description**: Get all notifications for the logged-in user  
- **Authentication**: Required  
- **Query Parameters**:  
  - isRead (boolean: true/false)  
  - page (number, default: 1)  
  - limit (number, default: 10)  
- **Response Success (200)**:   
  `{ success: true, data: { notifications: [...], pagination: { page, pages, total } } }`  
- **Response Errors**:  
  - 401: `{ success: false, error: "Authentication required" }`  
- **Potential User Mistakes**:  
  - Missing authentication token  

---

## PUT /api/notifications/:notificationId/read  
- **Method**: PUT  
- **Description**: Mark a notification as read  
- **Authentication**: Required (notification must belong to user)  
- **Request Body**: None  
- **Response Success (200)**:   
  `{ success: true, data: { id, isRead: true, updatedAt: "..." } }`  
- **Response Errors**:  
  - 401: `{ success: false, error: "Authentication required" }`  
  - 403: `{ success: false, error: "This notification does not belong to you" }`  
  - 404: `{ success: false, error: "Notification not found" }`  
- **Potential User Mistakes**:  
  - Attempting to mark another user’s notification as read  

---

## DELETE /api/projects/:projectId  
- **Method**: DELETE  
- **Description**: Delete a project (only by owner)  
- **Authentication**: Required  
- **Request Body**: None  
- **Response Success (200)**:   
  `{ success: true, message: "Project deleted successfully" }`  
- **Response Errors**:  
  - 401: `{ success: false, error: "Authentication required" }`  
  - 403: `{ success: false, error: "Only the project owner can delete this project" }`  
  - 404: `{ success: false, error: "Project not found" }`  
- **Potential User Mistakes**:  
  - Non-owner trying to delete a project  

---

## DELETE /api/tasks/:taskId  
- **Method**: DELETE  
- **Description**: Delete a task  
- **Authentication**: Required (creator or project admin)  
- **Request Body**: None  
- **Response Success (200)**:   
  `{ success: true, message: "Task deleted successfully" }`  
- **Response Errors**:  
  - 401: `{ success: false, error: "Authentication required" }`  
  - 403: `{ success: false, error: "You are not authorized to delete this task" }`  
  - 404: `{ success: false, error: "Task not found" }`  
- **Potential User Mistakes**:  
  - Trying to delete a task from a project they don’t belong to


  ## Middleware

### auth.middleware.js

**authenticate**  
- **Purpose**: Verify JWT token and attach user to the request object  
- **How it works**: Extracts token from `Authorization` header (Bearer scheme), verifies it using the JWT secret, and populates `req.user` with user ID. If token is missing, invalid, or expired, returns a 401 error.

**authorizeProjectAction**  
- **Purpose**: Ensure the authenticated user has permission to perform an action on a specific project  
- **How it works**: Checks if the user is either the project owner or a member with sufficient role (e.g., 'admin'). Used on routes like inviting members or deleting a project. Returns 403 if unauthorized.

**authorizeTaskAction**  
- **Purpose**: Validate that the user can view or modify a given task  
- **How it works**: Verifies the user is either the task creator, the assignee, or a member of the associated project. Prevents users from accessing or editing tasks outside their scope. Returns 403 on violation.

---

### error.middleware.js

**handleValidationErrors**  
- **Purpose**: Centralize validation error responses  
- **How it works**: Catches errors from `express-validator`, formats them into a user-friendly message array, and returns a 400 response with consistent structure: `{ success: false, error: "First validation error message" }`.

**notFound**  
- **Purpose**: Handle requests to undefined routes  
- **How it works**: Executes when no route matches the incoming request. Returns a 404 error: `{ success: false, error: "Route not found" }`.
---


## Authentication: JWT Implementation Strategy and Protected Routes

### JWT Implementation Strategy

**Token Structure**  
- **Payload**: Contains `userId` (from MongoDB `_id`), `email`, and `role`  
- **Algorithm**: HS256 (HMAC with SHA-256)  
- **Access Token Expiry**: 24 hours (`24h`) – balances security and user experience  
- **Secret Key**: Stored in environment variable (`JWT_SECRET`), at least 32 characters long  

**Authentication Flow**  
1. **Registration**:  
   - User submits `name`, `email`, `password`  
   - System hashes password with `bcryptjs` (salt rounds = 12)  
   - New user saved to `users` collection  
   - JWT access token generated and returned immediately  

2. **Login**:  
   - User provides `email` and `password`  
   - System finds user by email and compares hashed password  
   - On success, signs and returns JWT with user ID and role  

3. **Token Transmission**:  
   - Client stores token in **HTTP-only, secure cookie** (for web apps) **or**  
   - Client sends token in **`Authorization: Bearer <token>`** header (for mobile/API clients)  

4. **Token Verification**:  
   - `authenticate` middleware verifies token signature and expiry  
   - Decoded `userId` attached to `req.user.id` for downstream use  
   - Invalid/expired tokens return `401 Unauthorized`  

**Security Enhancements**  
- **Password Handling**:  
  - Never stored or logged in plain text  
  - Field excluded from all query results (`select: false` in Mongoose)  
- **Token Storage**:  
  - HTTP-only cookies prevent XSS-based token theft  
  - `SameSite=Strict` and `Secure` flags enforced in production  
- **Brute-Force Protection**:  
  - Rate limiting on `/login` and `/register` endpoints (5 attempts / 15 min per IP)  
- **Future-Proofing**:  
  - Design supports easy addition of refresh tokens and token invalidation (e.g., via Redis denylist)  

---

### Protected Routes

All routes below **require a valid JWT** in the `Authorization` header (or cookie). Access without authentication returns **401 Unauthorized**.

#### User Management (Self-access only)
- `GET /api/auth/me` – View own profile  
- *(Profile update/delete would also be protected and ownership-checked)*

#### Project Management
- `POST /api/projects` – Create a new project  
- `GET /api/projects` – List all projects the user belongs to  
- `POST /api/projects/:projectId/invite` – Invite members (requires project ownership/admin)  
- `DELETE /api/projects/:projectId` – Delete project (owner only)

#### Task Management
- `POST /api/tasks` – Create a task in a project  
- `GET /api/tasks` – View all personal and assigned tasks  
- `GET /api/tasks/me` – View only tasks assigned to the user  
- `PUT /api/tasks/:taskId` – Update task details or status  
- `DELETE /api/tasks/:taskId` – Delete a task (creator or project admin only)

#### Analytics
- `GET /api/analytics/tasks` – Access personal task completion statistics

#### Notifications
- `GET /api/notifications` – View own notifications  
- `PUT /api/notifications/:notificationId/read` – Mark notification as read



## Error Handling 

### Core Error Types
1. **Client Errors (4xx)**  
   → Caused by invalid user input or unauthorized actions.  
   **Examples**:  
   - `400`: Missing task title, invalid email, or past due date  
   - `401`: Accessing a protected route without login  
   - `403`: Trying to delete someone else’s project  
   - `404`: Requesting a task that doesn’t exist  

2. **Server Errors (5xx)**  
   → Unexpected system failures (never the user’s fault).  
   **Examples**:  
   - `500`: Database crash, unhandled code exception  

---

### How We Handle Errors
- **Consistent format**: Every error returns  
  ```json
  { "success": false, "error": "Clear message" }
  ```
- **User-friendly messages**: Never show technical details (e.g., stack traces).  
- **Early validation**: Reject bad requests immediately (e.g., check email format before saving).  
- **Log internally**: All errors are recorded for debugging—users never see raw logs.  
- **Fail safely**: If a non-critical feature fails (e.g., sending a notification), the main action (e.g., creating a task) still succeeds.

---

### Real Examples
| User Action | Error Response |
|-----------|----------------|
| Register with existing email | `400: "Email already exists"` |
| Create task without title | `400: "Title is required"` |
| Access `/api/tasks` without token | `401: "Authentication required"` |
| Assign task to user not in project | `400: "User is not a member of this project"` |
| Server database timeout | `500: "Server error"` (generic, safe message) |

