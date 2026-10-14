# Quizzer API

A quiz management REST API built with FastAPI, SQLAlchemy, and SQLite.

## Setup

```bash
pip install -r requirements.txt
uvicorn app.main:app --reload
```

For network access:
```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Base URL: `http://localhost:8000`
Interactive docs: `http://localhost:8000/docs`

---

## All Endpoints

### Authentication

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/auth/register` | No | Register new user |
| POST | `/api/auth/login` | No | Login and get token |
| GET | `/api/auth/me` | Yes | Get current user info |
| PUT | `/api/auth/me` | Yes | Update profile |
| POST | `/api/auth/logout` | Yes | Logout |

### Quizzes

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/quizzes/` | Yes | Create quiz |
| GET | `/api/quizzes/` | No | List all quizzes (deduplicated) |
| GET | `/api/quizzes/grouped` | No | Quizzes grouped by difficulty |
| GET | `/api/quizzes/my/attempts` | Yes | My quiz attempts with quiz details |
| GET | `/api/quizzes/{quiz_id}` | No | Get quiz detail |
| PUT | `/api/quizzes/{quiz_id}` | Yes | Update quiz (creator only) |
| DELETE | `/api/quizzes/{quiz_id}` | Yes | Delete quiz (creator only) |
| GET | `/api/quizzes/{quiz_id}/with-answers` | Yes | Get quiz with answers (creator only) |
| POST | `/api/quizzes/{quiz_id}/submit` | Yes | Submit quiz answers |
| GET | `/api/quizzes/{quiz_id}/leaderboard` | No | Quiz leaderboard |
| POST | `/api/quizzes/{quiz_id}/feedback` | Yes | Submit feedback for a quiz |
| GET | `/api/quizzes/{quiz_id}/feedback` | No | Get all feedback for a quiz |
| GET | `/api/quizzes/{quiz_id}/rating` | No | Get average rating of a quiz |

### Categories

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| POST | `/api/categories/` | No | Create category |
| GET | `/api/categories/` | No | List all categories |
| GET | `/api/categories/{category_id}` | No | Get category with subcategories |
| PUT | `/api/categories/{category_id}` | No | Update category |
| DELETE | `/api/categories/{category_id}` | No | Delete category |
| POST | `/api/categories/{category_id}/subcategories` | No | Create subcategory |
| GET | `/api/categories/{category_id}/subcategories` | No | List subcategories |
| PUT | `/api/subcategories/{subcategory_id}` | No | Update subcategory |
| DELETE | `/api/subcategories/{subcategory_id}` | No | Delete subcategory |

**Total: 24 endpoints**

---

## Available Categories and Subcategories

### Categories

| ID | Name | Description |
|----|------|-------------|
| 1 | Sports | Sports and athletics related quizzes |
| 2 | Technology | Technology and programming quizzes |
| 3 | Science | Science and nature quizzes |
| 4 | History | Historical events and figures |
| 5 | Entertainment | Movies, music, and pop culture |
| 6 | Geography | Geography and world knowledge quizzes |

### Subcategories

| ID | Category ID | Category | Name |
|----|-------------|----------|------|
| 1 | 1 | Sports | Cricket |
| 2 | 1 | Sports | Football |
| 3 | 1 | Sports | Basketball |
| 23 | 1 | Sports | Tennis |
| 4 | 2 | Technology | Python |
| 5 | 2 | Technology | JavaScript |
| 6 | 2 | Technology | Web Development |
| 7 | 3 | Science | Physics |
| 8 | 3 | Science | Biology |
| 9 | 4 | History | World War II |
| 10 | 5 | Entertainment | Movies |
| 21 | 6 | Geography | Countries |
| 22 | 6 | Geography | Capitals |

---

## Difficulty Levels

| Value | Description |
|-------|-------------|
| `easy` | Beginner level (default) |
| `medium` | Intermediate level |
| `hard` | Advanced level |

---

## Endpoint Details

### GET /api/auth/me

Header: `Authorization: Bearer <token>`

Returns the current authenticated user's profile.

```json
{
  "id": 10,
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "phone_number": "1234567890",
  "created_at": "2026-03-19T10:00:00"
}
```

---

### POST /api/auth/register

```json
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "phone_number": "1234567890",
  "password": "secret123"
}
```

Returns `201 Created`.

---

### POST /api/auth/login

```json
{
  "email": "john@example.com",
  "password": "secret123"
}
```

Response:
```json
{
  "access_token": "<jwt_token>",
  "token_type": "bearer",
  "user": {}
}
```

---


### PUT /api/auth/me

```json
{
  "first_name": "Jane",
  "last_name": "Doe",
  "phone_number": "9876543210"
}
```

---

### POST /api/quizzes/

```json
{
  "title": "Python Basics",
  "description": "Test your Python knowledge",
  "category_id": 2,
  "subcategory_id": 4,
  "difficulty": "easy",
  "time_limit": 30,
  "passing_score": 70,
  "questions": [
    {
      "question_text": "What is a list in Python?",
      "option_a": "A mutable sequence",
      "option_b": "An immutable sequence",
      "option_c": "A key-value store",
      "option_d": "A set of unique values",
      "correct_answer": "A",
      "difficulty": "easy"
    }
  ]
}
```

Returns `201 Created`.

---

### GET /api/quizzes/

Returns deduplicated quizzes — one per `(title, subcategory_id)`, latest record wins.

Optional filters: `?category_id=1`, `?subcategory_id=2`, `?difficulty=easy`, `?skip=0&limit=100`

---

### GET /api/quizzes/grouped

Returns quizzes grouped by difficulty, deduplicated by `(title, subcategory_id)`.

Optional: `?category_id=1&subcategory_id=2`

```json
{
  "easy": [],
  "medium": [],
  "hard": [],
  "easy_count": 3,
  "medium_count": 1,
  "hard_count": 0
}
```

---

### GET /api/quizzes/my/attempts

Returns all quiz attempts by the current user, including quiz details.

```json
[
  {
    "id": 5,
    "quiz_id": 7,
    "quiz_title": "Football Basics Quiz",
    "quiz_category": "Sports",
    "quiz_subcategory": "Football",
    "quiz_difficulty": "easy",
    "user_id": 10,
    "score": 80,
    "correct_answers": 4,
    "total_questions": 5,
    "time_taken": 900,
    "time_limit": 30,
    "time_exceeded": false,
    "passed": true,
    "completed_at": "2026-03-19T10:00:00"
  }
]
```

---

### POST /api/quizzes/{quiz_id}/submit

`time_taken` in seconds. `time_limit` on quiz is in minutes.

```json
{
  "answers": [
    { "question_id": 1, "selected_answer": "A" }
  ],
  "time_taken": 900
}
```

Normal response:
```json
{
  "score": 80,
  "passed": true,
  "time_taken": 900,
  "time_limit": 30,
  "time_exceeded": false,
  "correct_answers": 4,
  "total_questions": 5,
  "quiz_category": "Technology",
  "quiz_subcategory": "Python",
  "quiz_difficulty": "easy"
}
```

Time exceeded response:
```json
{
  "score": 0,
  "passed": false,
  "time_exceeded": true
}
```

---

### GET /api/quizzes/{quiz_id}/leaderboard

Optional: `?limit=10`

One entry per user, best attempt (highest score then lowest time), excludes score = 0.

```json
[
  {
    "rank": 1,
    "user_id": 5,
    "user_name": "John Doe",
    "score": 95,
    "time_taken": 540,
    "completed_at": "2026-03-17T10:00:00"
  }
]
```

---

### POST /api/categories/

```json
{ "name": "Sports", "description": "Sports quizzes" }
```

---

### POST /api/categories/{category_id}/subcategories

```json
{ "name": "Football", "description": "Football quizzes" }
```

---

## Question-Level Difficulty (New Feature)

Each question can have its own difficulty independent of the quiz-level difficulty.

| Field | Type | Values | Default |
|-------|------|--------|---------|
| `difficulty` | string | `easy`, `medium`, `hard` | `easy` |
| `points` | int | 1-10 | auto from difficulty |

If `points` is not provided, it is auto-set:

| Difficulty | Auto Points |
|------------|-------------|
| `easy` | 1 |
| `medium` | 2 |
| `hard` | 3 |

Example with mixed difficulty questions:
```json
{
  "title": "Python Advanced Quiz",
  "category_id": 2,
  "subcategory_id": 4,
  "difficulty": "medium",
  "time_limit": 30,
  "passing_score": 70,
  "questions": [
    {
      "question_text": "What is a list in Python?",
      "option_a": "A mutable sequence",
      "option_b": "An immutable sequence",
      "option_c": "A key-value store",
      "option_d": "A set of unique values",
      "correct_answer": "A",
      "difficulty": "easy"
    },
    {
      "question_text": "What does the GIL stand for in Python?",
      "option_a": "Global Interpreter Lock",
      "option_b": "General Input Loop",
      "option_c": "Global Index List",
      "option_d": "General Interpreter Layer",
      "correct_answer": "A",
      "difficulty": "medium"
    },
    {
      "question_text": "What is the time complexity of dict lookup in Python?",
      "option_a": "O(n)",
      "option_b": "O(log n)",
      "option_c": "O(1)",
      "option_d": "O(n^2)",
      "correct_answer": "C",
      "difficulty": "hard"
    }
  ]
}
```

---

## Data Validation and Consistency

### Category and Subcategory Mismatch

When creating or updating a quiz, `subcategory_id` must belong to the given `category_id`.

If they don't match:
```json
{ "detail": "Category and Subcategory mismatch" }
```
Status: `400 Bad Request`

### Question ID Validation on Submit

All submitted `question_id` values must belong to the quiz being submitted. Invalid IDs return:
```json
{ "detail": "Question IDs [99] do not belong to quiz 7" }
```

---

---

## Feedback System

### POST /api/quizzes/{quiz_id}/feedback

Header: `Authorization: Bearer <token>`

Submit feedback for a quiz. One feedback per user per quiz.

```json
{
  "rating": 4,
  "comment": "Great quiz, very informative!"
}
```

`rating` must be between 1 and 5. `comment` is optional.

Response `201 Created`:
```json
{
  "id": 1,
  "quiz_id": 7,
  "user_id": 10,
  "rating": 4,
  "comment": "Great quiz, very informative!",
  "created_at": "2026-03-20T10:00:00"
}
```

If already submitted: `400 You have already submitted feedback for this quiz`

---

### GET /api/quizzes/{quiz_id}/feedback

Returns all feedback for a quiz, latest first.

```json
[
  {
    "id": 1,
    "quiz_id": 7,
    "user_id": 10,
    "rating": 4,
    "comment": "Great quiz!",
    "created_at": "2026-03-20T10:00:00"
  }
]
```

---

### GET /api/quizzes/{quiz_id}/rating

Returns average rating and total count for a quiz.

```json
{
  "quiz_id": 7,
  "average_rating": 4.25,
  "total_ratings": 8
}
```

---

## Timer Enforcement

If `time_taken > time_limit * 60`: score is forced to 0, passed is false, time_exceeded is true. Attempt is saved but excluded from leaderboard.

---

## Deduplication

Both `GET /api/quizzes/` and `GET /api/quizzes/grouped` deduplicate by `(title, subcategory_id)`.
Uses a SQL subquery with `max(id)` to select the latest record per group.
Filters are applied before grouping. Pagination (`skip`/`limit`) is applied after deduplication.
