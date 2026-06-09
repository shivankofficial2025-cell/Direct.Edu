# DirectEdu - API Documentation

## Base URL

```
Development: http://localhost:5000/api/v1
Production: https://api.directedu.com/api/v1
```

## Authentication

### Login
```
POST /auth/login
```

**Request:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "first_name": "John",
      "last_name": "Doe",
      "role": "student"
    },
    "tokens": {
      "access_token": "jwt_token",
      "refresh_token": "refresh_token",
      "expires_in": 3600
    }
  }
}
```

## Student Endpoints

### Get Dashboard
```
GET /students/dashboard
```

### Get Assignments
```
GET /students/:student_id/assignments
```

### Submit Assignment
```
POST /students/:student_id/assignments/:assignment_id/submit
```

### Get Grades
```
GET /students/:student_id/grades
```

### Get Attendance
```
GET /students/:student_id/attendance
```

## Teacher Endpoints

### Get Classes
```
GET /teachers/:teacher_id/classes
```

### Create Assignment
```
POST /classes/:class_id/assignments
```

### Grade Assignment
```
PUT /assignments/:assignment_id/submissions/:submission_id/grade
```

### Record Attendance
```
POST /classes/:class_id/attendance
```

## Administrator Endpoints

### User Management
```
GET /admin/users
POST /admin/users
PUT /admin/users/:user_id
DELETE /admin/users/:user_id
```

### Analytics
```
GET /admin/analytics/school
GET /admin/analytics/students
GET /admin/analytics/teachers
```

---

**Version**: 1.0.0