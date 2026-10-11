# TaskFlow

A full-stack task manager built with the MERN stack. Users register, sign in, and manage their own tasks with status, priority, description, and due date. Each user only ever sees and changes their own data.

Repository: <https://github.com/Rahulkalakoti45/taskflow-mern>

## Key features

- **Authentication**: register and sign in with email and password. Passwords are hashed with bcrypt, and sessions use JWTs sent as bearer tokens.
- **Task management**: create, edit, delete, and move tasks through `To do → In progress → Done`.
- **Filtering and summary**: filter the list by status, with counts per status shown at the top.
- **Due dates and overdue highlighting**: tasks past their due date are flagged until they are done.
- **Ownership enforcement**: every task query is scoped to the authenticated user. Cross-user access returns `404`.
- **Input validation**: server-side validation returns field-level errors, which the UI displays.
- **Responsive UI**: usable on phone-width screens with a 16px gutter and no horizontal scrolling.

## Technologies used

| Layer    | Stack                                                                 |
|----------|-----------------------------------------------------------------------|
| Database | MongoDB with Mongoose 8                                               |
| Backend  | Node.js, Express 4, express-validator, jsonwebtoken, bcryptjs, cors   |
| Frontend | React 18, React Router 6, Vite 5                                      |
| Testing  | Node's built-in test runner, Supertest, mongodb-memory-server         |

## Project structure

```
taskflow-mern/
├── server/                 # Express REST API
│   ├── src/
│   │   ├── app.js          # Express app factory (no listener; testable)
│   │   ├── server.js       # Entry point: loads env, connects DB, listens
│   │   ├── config/db.js    # Mongoose connection helper
│   │   ├── models/         # User and Task schemas
│   │   ├── middleware/     # JWT auth, validation, error handling
│   │   ├── controllers/    # Request handlers
│   │   └── routes/         # Route definitions and validation rules
│   └── test/               # Integration tests
└── client/                 # React SPA (Vite)
    └── src/
        ├── api/            # fetch wrapper and service functions
        ├── context/        # AuthContext (session state)
        ├── components/     # TaskForm, TaskItem, ProtectedRoute
        └── pages/          # AuthPage (login/register), DashboardPage
```

## Setup and installation

**Prerequisites:** Node.js 18+ and npm, plus MongoDB running locally (`mongodb://127.0.0.1:27017`) or a MongoDB Atlas connection string.

```bash
git clone https://github.com/Rahulkalakoti45/taskflow-mern.git
cd taskflow-mern

# Install dependencies
cd server && npm install
cd ../client && npm install
cd ..

# Configure the API
cp server/.env.example server/.env
```

Then open `server/.env` and set `JWT_SECRET` to a long random string.

## Running the application

Open two terminals.

```bash
# Terminal 1: API on http://localhost:5000
cd server
npm run dev        # or: npm start
```

```bash
# Terminal 2: UI on http://localhost:5173
cd client
npm run dev
```

Open <http://localhost:5173>, register an account, and start adding tasks. Vite proxies `/api` requests to the Express server, so no CORS configuration is needed in development.

To build the production bundle:

```bash
cd client && npm run build
```

## Running the tests

```bash
cd server
npm test
```

The tests start an in-memory MongoDB instance automatically, so no local database is needed. They cover registration and login, token enforcement, task CRUD, cross-user access control, validation errors, and filtering.

## REST API

All task endpoints require `Authorization: Bearer <token>`.

| Method | Path                 | Description                                          |
|--------|----------------------|------------------------------------------------------|
| GET    | `/api/health`        | Health check                                         |
| POST   | `/api/auth/register` | Create account, returns `{ token, user }`            |
| POST   | `/api/auth/login`    | Sign in, returns `{ token, user }`                   |
| GET    | `/api/auth/me`       | Current user                                         |
| GET    | `/api/tasks`         | List own tasks; optional `?status=` and `?priority=` |
| POST   | `/api/tasks`         | Create a task                                        |
| PATCH  | `/api/tasks/:id`     | Partially update a task                              |
| DELETE | `/api/tasks/:id`     | Delete a task (returns `204`)                        |

## Environment variables

| Variable         | Default                              | Purpose                   |
|------------------|--------------------------------------|---------------------------|
| `PORT`           | `5000`                               | API port                  |
| `MONGO_URI`      | `mongodb://127.0.0.1:27017/taskflow` | MongoDB connection string |
| `JWT_SECRET`     | *(required)*                         | Signing key for JWTs      |
| `JWT_EXPIRES_IN` | `7d`                                 | Token lifetime            |
| `CLIENT_ORIGIN`  | `http://localhost:5173`              | Allowed CORS origin       |

## Design decisions

- **App factory (`createApp`)** separates building the Express app from listening on a port, so tests can mount it with Supertest without opening a socket.
- **Ownership is enforced in the query**, not after the fact. `findOneAndUpdate({ _id, owner })` means a user who guesses another user's task id gets a `404` and nothing changes.
- **`passwordHash` is `select: false`** and removed in `toJSON`, so it can't leak through a future query or response by accident.
- **Validation at the edge, types in the model.** express-validator returns a readable field map; Mongoose enums and `maxlength` stay as a second line of defence.
- **Stateless JWT auth.** Simple to scale horizontally. The trade-off is that tokens can't be revoked before they expire, which is acceptable for this scope.
- **Optimistic status changes in the UI** make the status button feel instant. The previous list is restored if the request fails.
- **Dates are handled in UTC**, since they are stored as UTC midnight. The overdue check and the display therefore agree in every time zone.

## Development with Code0

This project was built with **Code0**, which I used across the full stack, from the Express API and database layer to the React frontend and documentation.

### AI Development Experience

I worked with Code0 as a pair programmer. I described each feature or problem in plain language, Code0 proposed code across the server, database, and client layers, and I reviewed each change, ran it, and iterated until it behaved correctly. I checked the security-sensitive parts myself, including ownership filters and password handling.

### Where Code0 helped

1. **API creation**: scaffolded the Express app factory, the JWT auth middleware, and the task routes with express-validator rules.
2. **Database integration**: wired Mongoose `User` and `Task` schemas to MongoDB, including the `passwordHash` field hidden by default and per-user scoping on every task query.
3. **Component development**: built the React pages and components, including `TaskForm`, `TaskItem`, `ProtectedRoute`, the login and registration flow, and the dashboard.
4. **Testing and debugging**: generated Supertest integration tests against `mongodb-memory-server` and fixed a test-runner setup that failed on newer Node versions.
5. **Refactoring**: moved the due-date logic to UTC to prevent off-by-one-day errors across time zones, and replaced an unclear `slice(1)` in the task routes with named validation rules.

## License

MIT
