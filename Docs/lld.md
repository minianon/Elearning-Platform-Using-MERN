# E-learning Platform (MERN): Low-Level Design

For the big picture see the [High-Level Design](hld.md).

## 1. Models

```mermaid
erDiagram
  USER }o--o{ COURSE : enrolls
  COURSE ||--o{ VIDEO : contains
  USER {
    string name
    string email
    string password
    number age
    string city
    string job
    array course
    string role "student | teacher | admin"
    string image
    bool isPromotion
  }
  COURSE {
    string title
    string description
    string category
    string teacher
    string teacherId
    number price
    string img
    array videos
  }
  VIDEO {
    string title
    string description
    string teacher
    string teacherId
    number views
    string link
    number video_length
    string img
    ObjectId courseId
  }
  BLACKLIST {
    string token
  }
```

## 2. API

| Router | Routes |
|---|---|
| `/users` | `GET /`, `POST /register`, `POST /login`, `PATCH /update/:userId`, `DELETE /delete/:userId`, `POST /logout`, `GET /userCourse/:userId`, `POST /addCourse/:courseId`, `GET /Teachme/:userId` |
| `/courses` | `GET /all`, `GET /`, `GET /TeacherCourses`, `GET /:courseID`, `POST /add`, `PATCH /update/:courseID`, `DELETE /delete/:courseID` |
| `/videos` | `GET /`, `GET /:videoID`, `POST /add/:courseId`, `GET /courseVideos/:courseId` |
| root | `GET /regenerateToken` (refresh → new access token) |

## 3. Auth flow

```mermaid
sequenceDiagram
  actor U as User
  participant FE as Front end
  participant API as Express
  participant DB as MongoDB
  U->>FE: login
  FE->>API: POST /users/login
  API->>DB: find user, bcrypt.compare
  API-->>FE: access token (7 d) + refresh token (24 d)
  FE->>API: requests with Bearer access token
  API->>API: middleware: reject blacklisted token, jwt.verify, attach userId and role
  FE->>API: GET /regenerateToken (Bearer refresh token)
  API-->>FE: new access token
  FE->>API: POST /users/logout
  API->>DB: add token to blacklist
```

## 4. Enrollment

The payment page shows a card form, then calls `POST /users/addCourse/:courseId`, which adds the course to the user's list. No payment provider is connected, so it works as a mock checkout.

## 5. Known issues

| Issue | Risk | Fix |
|---|---|---|
| JWT signing secrets are hard-coded string literals in the source | Anyone who reads the public repo can forge tokens for any user or role | Move them to environment variables and rotate them |
| Access and refresh tokens use different secrets in different files | Easy to break one path when changing the other | One config module for both |
| Blacklist is checked from an in-memory list and the database | Can disagree after restarts | Check only the database (or use short-lived tokens) |
