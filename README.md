# StaffFlow – Employee Management System (EMS) & REST API

StaffFlow is an enterprise-grade, full-stack Employee Management System built with a Node.js / Express TypeScript backend, a responsive React 19 + Tailwind CSS frontend, and a relational database engine with foreign key enforcement and indexed queries.

---

## 🌟 Key Features

### 1. Authentication & Security
- **JWT-Based Authentication**: Secure stateless token issuance and verification with expiration handling.
- **bcrypt Password Hashing**: Adaptive hashing with salt rounds for user credentials.
- **Role-Based Access Control (RBAC)**: Distinct permissions for `admin`, `hr_manager`, and `viewer` roles.
- **Input Validation & Sanitization**: Zod validation schemas for all incoming HTTP request bodies and query parameters.
- **SQL Injection Prevention**: 100% parameterized queries across all database read and write operations.

### 2. Executive Analytics Dashboard
- **Workforce KPIs**: Real-time counters for Total Employees, Active Staff, Inactive/On-Leave, and Operating Departments.
- **Financial Metrics**: Total annual active payroll and average base compensation.
- **Department Distribution**: Interactive SVG distribution charts displaying staffing counts, salary allocations, and percentages.
- **Hiring Velocity Trends**: Quarterly talent onboarding timeline visualization.
- **Recent Additions**: Direct access to recently onboarded team members.

### 3. Comprehensive Employee CRUD Management
- **Full Profile Lifecycle**:
  - Employee ID (auto-generated e.g. `EMP-1001` or custom defined).
  - Personal details: Full Name, Gender, Date of Birth.
  - Job assignments: Department, Designation / Job Title, Joining Date.
  - Compensation & Status: Annual Base Salary, Employment Status (`active`, `inactive`, `on_leave`).
  - Contact & Address: Work Email, Phone Number, Residential Address.
- **Search & Filtering**:
  - Substring search across name, email, employee ID, and job designation.
  - Department filtering dropdown.
  - Employment status filtering (`active`, `inactive`, `on_leave`, `all`).
  - Column sorting by Name, Joining Date, Salary, or Creation Date (`asc`/`desc`).
- **Pagination**: Configurable items-per-page (5, 10, 25, 50) and page navigation.
- **Safe Deletion Guard**: Two-step confirmation modal with item name matching to prevent accidental removals.
- **Data Export**: One-click full dataset export to CSV.

### 4. Department Management
- Create, view, edit, and delete business units with assigned department managers and budgets.
- **Relational Integrity Protection**: Foreign key protection prevents deletion of departments with assigned active employees.
- **Assigned Team Modal**: Inspect all employees belonging to any department in a single click.

### 5. API Documentation & Automated Testing
- **Swagger / OpenAPI 3.0**: Interactive UI served at `/api/docs` and JSON schema at `/api/docs.json`.
- **Postman Collection v2.1**: Complete collection with pre-configured request scripts and tests in `/postman/Employee_Management_System.postman_collection.json`.
- **In-App Automated Test Runner**: One-click execution of backend integration tests directly from the web interface.

---

## 🏗️ Technology Stack

| Layer | Technologies |
|---|---|
| **Frontend** | React 19, TypeScript, Vite, Tailwind CSS v4, React Router v7, React Hook Form, Axios, Lucide Icons |
| **Backend** | Node.js, Express.js, TypeScript, TSX, JWT (`jsonwebtoken`), `bcryptjs`, Zod |
| **Database** | MySQL 8.0 / MariaDB with `mysql2/promise` connection pooling, InnoDB foreign keys, and indexes |
| **Documentation** | OpenAPI 3.0, Swagger UI Express, Postman Collection v2.1 |

---

## 🏛️ Project Architecture

```
.
├── database/
│   ├── schema.sql           # MySQL 8.0 production DDL schema
│   └── seed.sql             # Sample departments, admin, and employees
├── postman/
│   └── Employee_Management_System.postman_collection.json
├── src/
│   ├── config/              # Runtime env, DB pool, and Swagger config
│   │   ├── env.ts
│   │   ├── db.ts
│   │   └── swagger.ts
│   ├── controllers/         # HTTP controllers
│   │   ├── authController.ts
│   │   ├── employeeController.ts
│   │   ├── departmentController.ts
│   │   └── dashboardController.ts
│   ├── routes/              # Express API routers
│   │   ├── authRoutes.ts
│   │   ├── employeeRoutes.ts
│   │   ├── departmentRoutes.ts
│   │   ├── dashboardRoutes.ts
│   │   └── testRoutes.ts
│   ├── middleware/          # JWT auth, Zod validation, error handlers
│   │   ├── authMiddleware.ts
│   │   ├── validateMiddleware.ts
│   │   └── errorMiddleware.ts
│   ├── services/            # Business logic and database operations
│   │   ├── authService.ts
│   │   ├── employeeService.ts
│   │   ├── departmentService.ts
│   │   ├── dashboardService.ts
│   │   └── clientServices.ts  # Frontend typed Axios client
│   ├── validators/          # Zod request validation schemas
│   ├── types/               # TypeScript interfaces & data contracts
│   ├── tests/               # Automated unit & integration tests
│   │   └── runTests.ts
│   ├── components/          # Reusable UI widgets & modals
│   ├── pages/               # Routed pages (Dashboard, Employees, etc.)
│   ├── layouts/             # AppLayout shell
│   ├── context/             # AuthContext and ToastContext
│   └── utils/               # Formatting, passwords, JWT, CSV
├── server.ts                # Unified Express entry point with Vite middlewares
├── index.html
├── package.json
└── tsconfig.json
```

---

## 🗄️ Database Schema Design

### 1. `users` Table
```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(100) NOT NULL,
    role VARCHAR(20) NOT NULL DEFAULT 'admin',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_users_email (email),
    INDEX idx_users_username (username)
);
```

### 2. `departments` Table
```sql
CREATE TABLE departments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL UNIQUE,
    description TEXT,
    manager_name VARCHAR(100),
    budget DECIMAL(12, 2) DEFAULT 0.00,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_departments_name (name)
);
```

### 3. `employees` Table
```sql
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    employee_id VARCHAR(20) NOT NULL UNIQUE,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    phone VARCHAR(30) NOT NULL,
    gender VARCHAR(20) NOT NULL,
    date_of_birth DATE NOT NULL,
    department_id INT NOT NULL,
    designation VARCHAR(100) NOT NULL,
    joining_date DATE NOT NULL,
    salary DECIMAL(12, 2) NOT NULL DEFAULT 0.00,
    status VARCHAR(20) NOT NULL DEFAULT 'active',
    address TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    CONSTRAINT fk_employees_department 
        FOREIGN KEY (department_id) 
        REFERENCES departments (id) 
        ON DELETE RESTRICT 
        ON UPDATE CASCADE,
    INDEX idx_employees_department_id (department_id),
    INDEX idx_employees_status (status),
    INDEX idx_employees_email (email),
    INDEX idx_employees_full_name (full_name),
    INDEX idx_employees_joining_date (joining_date)
);
```

---

## 🚀 Quickstart & Running Locally

### 1. Prerequisites
- Node.js >= 18.0.0
- npm >= 9.0.0

### 2. Environment Setup
Copy the example environment file:
```bash
cp .env.example .env
```

Default values in `.env`:
```env
PORT=3000
NODE_ENV=development
JWT_SECRET=employee-management-super-secret-jwt-key-2026
JWT_EXPIRES_IN=24h
```

### 3. Installation
Install all dependencies:
```bash
npm install
```

### 4. Run Development Server
Start the Express server with Vite frontend mounted:
```bash
npm run dev
```
Open **`http://localhost:3000`** in your browser.

---

## 🔑 Default Credentials

| Role | Email | Password |
|---|---|---|
| **Super Admin** | `admin@staffflow.com` | `admin123` |
| **HR Manager** | `hr@staffflow.com` | `admin123` |

*(You can also click the quick one-click demo buttons on the login screen to autofill credentials).*

---

## 📡 REST API Reference

All protected endpoints require the HTTP header:
`Authorization: Bearer <your_jwt_token>`

### Authentication
- `POST /api/auth/login` - Authenticate with email/username and password.
- `POST /api/auth/logout` - Invalidate active session.
- `GET /api/auth/profile` - Fetch current user profile.

### Employees
- `GET /api/employees` - List employees with search, department filter, status filter, and pagination (`page`, `limit`, `sortBy`, `sortOrder`).
- `GET /api/employees/:id` - Get employee details by ID.
- `POST /api/employees` - Create a new employee record.
- `PUT /api/employees/:id` - Update employee information.
- `DELETE /api/employees/:id` - Delete an employee record.

### Departments
- `GET /api/departments` - List all departments with headcount totals.
- `GET /api/departments/:id` - Get department info and assigned employees.
- `POST /api/departments` - Create a new department.
- `PUT /api/departments/:id` - Update department details.
- `DELETE /api/departments/:id` - Delete department (enforces 0 assigned employees).

### Dashboard & Analytics
- `GET /api/dashboard/stats` - Fetch KPIs, department distribution, and joining trends.
- `POST /api/dashboard/reset` - Reset database to factory demo state.

---

## 🧪 Testing

Run automated tests via CLI:
```bash
npx tsx src/tests/runTests.ts
```

Or run the tests directly in the browser by visiting **`/api-docs`** and clicking **"Run Automated Tests"**.

---

## 🚢 Deployment Instructions

### Production Build
```bash
npm run build
npm start
```

### Deploying to Render / Railway / Replit
1. Connect your Git repository.
2. Build Command: `npm install && npm run build`
3. Start Command: `npm start`
4. Set Environment Variables:
   - `NODE_ENV=production`
   - `JWT_SECRET=<strong-random-secret>`
   - `PORT=3000`
