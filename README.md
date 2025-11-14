# 📘 Habit Tracker — Requirements Specification

## 1. **System Overview**

The system allows a single user (no authentication for MVP) to create habits and log daily completions. The frontend provides an interface to manage habits and visualize progress. The backend exposes REST APIs for CRUD operations and daily logging. Data is stored in SQLite, with schema designed to be compatible with PostgreSQL.

## 2. **Functional Requirements**

### 2.1 Habits

#### **FR-1: Create Habit**

- User can create a habit with:
  - name (required)
  - description (optional)

#### **FR-2: View All Habits**

- User can fetch a list of all habits.
- Each habit includes:
  - id
  - name
  - description
  - created_at

#### **FR-3: Update Habit**

- User can edit the habit name or description.

#### **FR-4: Delete Habit**

- User can delete a habit.
- Deleting a habit also deletes associated logs.

### 2.2 Habit Logs

#### **FR-5: Log Habit Completion**

- User can mark a habit as completed for the current date.
- A habit can only be marked completed **once per date**.

#### **FR-6: Undo Habit Completion**

- User can unmark a habit for a specific date (delete log).

#### **FR-7: View Habit Logs**

- User can fetch logs for a habit, optionally filtered by:
  - `date=YYYY-MM-DD`
  - `from` and `to` date range
- Logs include:
  - date (YYYY-MM-DD)
  - habit_id

#### **FR-8: 7-Day Summary**

- User can fetch the habit’s last 7 days completion status:
  - Useful for a frontend chart
  - Returns an array:
    ```json
    [
      { "date": "2025-01-01", "completed": true },
      { "date": "2025-01-02", "completed": false },
      ...
    ]
    ```

## 3. **Non-Functional Requirements**

### 3.1 Performance

- The system stores small datasets; all endpoints should respond <200ms locally.

### 3.2 REST Architecture

- Use proper HTTP methods (GET/POST/PUT/DELETE).
- Consistent JSON responses.
- Use status codes:
  - `200 OK`
  - `201 Created`
  - `204 No Content`
  - `400 Bad Request`
  - `404 Not Found`
  - `409 Conflict` (e.g., double marking a habit in a day)

### 3.3 Testing Strategy (Test Pyramid)

#### **Unit Tests**

- Pure logic:
  - date utilities
  - validation functions
  - repository interfaces (mock behavior)
  - summary calculation

#### **Integration Tests**

- Gin handlers using SQLite (in-memory)
- Endpoints:
  - habit CRUD
  - logging endpoints
  - 7-day summary
- Error scenarios:
  - duplicate log
  - deleting non-existing habit
  - updating non-existing habit

#### **E2E Tests**

- Using Playwright on the frontend:
  - Create habit → appears in list
  - Mark habit complete → UI updates
  - Undo completion → UI updates
  - Delete habit → disappears

### 3.4 Migration Readiness (SQLite → PostgreSQL)

- Use integer primary keys (autoincrement works for both)
- Use `created_at` timestamps in ISO format
- Avoid SQLite-specific features
- Use raw SQL with standardized schema

## 4. **Frontend Requirements (Next.js + TypeScript)**

### 4.1 Habit Management

- Page to list all habits.
- Form to create a new habit.
- Ability to edit and delete habits.

### 4.2 Logging Interaction

- Button to mark today as completed.
- Button to undo log for today.
- Visual indicator: completed vs not completed today.

### 4.3 7-Day Summary Chart

- Display a basic bar or line chart.
- No complex charting library needed; simple canvas or lightweight chart lib (e.g., Chart.js).

## 5. **Backend Requirements (Go + Gin)**

### 5.1 Endpoints

#### Habits

| Method | Path        | Description         |
| ------ | ----------- | ------------------- |
| POST   | /habits     | Create habit        |
| GET    | /habits     | List all habits     |
| GET    | /habits/:id | Get habit by ID     |
| PUT    | /habits/:id | Update habit        |
| DELETE | /habits/:id | Delete habit + logs |

#### Logs

| Method | Path                   | Description                       |
| ------ | ---------------------- | --------------------------------- |
| POST   | /habits/:id/logs       | Mark complete for today           |
| DELETE | /habits/:id/logs/:date | Undo completion for given date    |
| GET    | /habits/:id/logs       | List logs (with optional filters) |
| GET    | /habits/:id/summary7   | 7-day completion summary          |

## 6. **Data Model Requirements**

### 6.1 Habit

- `id`: integer PK
- `name`: text (required)
- `description`: text (optional)
- `created_at`: timestamp

### 6.2 HabitLog

- `habit_id`: FK to Habit
- `date`: text (YYYY-MM-DD)
- Primary key: (`habit_id`, `date`)

## 7. **Out of Scope for This Project**

- User authentication
- Multi-user support
- Mobile application
- Advanced statistics/graphs
- Notifications or reminders
