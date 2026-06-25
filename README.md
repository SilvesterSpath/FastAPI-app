# TodoApp API

A FastAPI todo application with user authentication, role-based access, and SQLite persistence.

## Features

- User registration and JWT login
- CRUD for todos (scoped to the logged-in user)
- Password change for authenticated users
- Admin endpoints to view and delete any todo

## Tech stack

- [FastAPI](https://fastapi.tiangolo.com/)
- [SQLAlchemy](https://www.sqlalchemy.org/) + SQLite
- [Pydantic](https://docs.pydantic.dev/) for request validation
- JWT auth via `python-jose`
- Password hashing via `passlib` + `bcrypt`

## Project structure

```
TodoApp/
├── main.py           # App entry point, creates DB tables on startup
├── database.py       # SQLite engine and session setup
├── models.py         # SQLAlchemy models (Users, Todos)
├── routers/
│   ├── auth.py       # Register, login, JWT helpers
│   ├── todos.py      # Todo CRUD for authenticated users
│   ├── users.py      # User profile and password change
│   └── admin.py      # Admin-only todo endpoints
├── requirements.txt
└── todosapp.db       # Created locally at runtime (not committed)
```

## Setup

1. Create and activate a virtual environment:

   ```bash
   python -m venv .venv

   # Windows
   .venv\Scripts\activate

   # macOS / Linux
   source .venv/bin/activate
   ```

2. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

3. Run the app from the `TodoApp` directory:

   ```bash
   uvicorn main:app --reload
   ```

4. Open the interactive API docs:

   - Swagger UI: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
   - ReDoc: [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)

On first run, `main.py` creates `todosapp.db` and the required tables automatically.

## Quick start (API flow)

### 1. Register a user

`POST /auth/`

```json
{
  "username": "johndoe",
  "email": "john@example.com",
  "first_name": "John",
  "last_name": "Doe",
  "password": "secretpassword",
  "role": "admin"
}
```

Use `"role": "admin"` only for admin endpoints. Regular users can use any other role value.

### 2. Log in and get a token

`POST /auth/token` (form-urlencoded)

- `username`: your username
- `password`: your password

Response:

```json
{
  "access_token": "...",
  "token_type": "bearer"
}
```

### 3. Call protected routes

In Swagger UI, click **Authorize** and paste:

```text
Bearer <your_access_token>
```

Or send the header manually:

```text
Authorization: Bearer <your_access_token>
```

### 4. Todo endpoints (authenticated user)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/` | List your todos |
| `GET` | `/todo/{todo_id}` | Get one todo |
| `POST` | `/todo` | Create a todo |
| `PUT` | `/todo/{todo_id}` | Update a todo |
| `DELETE` | `/todo/{todo_id}` | Delete a todo |

Example create body:

```json
{
  "title": "Buy groceries",
  "description": "Milk and eggs",
  "priority": 3,
  "complete": false
}
```

### 5. User endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/user/` | Get current user profile |
| `PUT` | `/user/password` | Change password |

### 6. Admin endpoints

Requires a user with `role: "admin"`.

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/admin/todo` | List all todos |
| `DELETE` | `/admin/todo/{todo_id}` | Delete any todo |

## Viewing the database

`todosapp.db` is a local SQLite file. You can inspect it with [DB Browser for SQLite](https://sqlitebrowser.org/) or any SQLite GUI.

## Notes

- `todosapp.db` is ignored by git; each environment creates its own database.
- The JWT secret in `routers/auth.py` is for learning only. Use environment variables for a real deployment.
