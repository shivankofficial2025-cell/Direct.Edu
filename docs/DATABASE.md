# DirectEdu - Database Schema

## Overview

This document outlines the complete PostgreSQL database schema for DirectEdu.

## Core Tables

### Users
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  phone_number VARCHAR(20),
  date_of_birth DATE,
  gender VARCHAR(20),
  profile_picture_url TEXT,
  password_hash VARCHAR(255) NOT NULL,
  mfa_enabled BOOLEAN DEFAULT FALSE,
  mfa_secret VARCHAR(255),
  last_login TIMESTAMP,
  login_attempts INT DEFAULT 0,
  locked_until TIMESTAMP,
  role VARCHAR(50) NOT NULL,
  status VARCHAR(20) DEFAULT 'active',
  is_verified BOOLEAN DEFAULT FALSE,
  email_verified_at TIMESTAMP,
  institution_id UUID NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP,
  FOREIGN KEY (institution_id) REFERENCES institutions(id),
  INDEX idx_email (email),
  INDEX idx_institution (institution_id),
  INDEX idx_role (role)
);
```

### Institutions
```sql
CREATE TABLE institutions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL UNIQUE,
  slug VARCHAR(255) UNIQUE,
  description TEXT,
  logo_url TEXT,
  address_line1 VARCHAR(255),
  address_line2 VARCHAR(255),
  city VARCHAR(100),
  state VARCHAR(100),
  postal_code VARCHAR(20),
  country VARCHAR(100),
  email VARCHAR(255),
  phone_number VARCHAR(20),
  website_url VARCHAR(255),
  academic_year_start DATE,
  academic_year_end DATE,
  working_days_per_week INT DEFAULT 5,
  school_start_time TIME,
  school_end_time TIME,
  subscription_tier VARCHAR(50),
  subscription_status VARCHAR(20),
  subscription_expires_at TIMESTAMP,
  max_users INT DEFAULT 100,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  deleted_at TIMESTAMP,
  INDEX idx_slug (slug),
  INDEX idx_subscription_status (subscription_status)
);
```

### Classes & Enrollments
```sql
CREATE TABLE classes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  institution_id UUID NOT NULL,
  name VARCHAR(100) NOT NULL,
  section VARCHAR(50),
  level INT,
  description TEXT,
  capacity INT DEFAULT 50,
  current_strength INT DEFAULT 0,
  class_teacher_id UUID,
  academic_year_id UUID NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (institution_id) REFERENCES institutions(id),
  FOREIGN KEY (class_teacher_id) REFERENCES users(id),
  FOREIGN KEY (academic_year_id) REFERENCES academic_years(id),
  INDEX idx_institution_year (institution_id, academic_year_id)
);

CREATE TABLE enrollments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID NOT NULL,
  class_id UUID NOT NULL,
  academic_year_id UUID NOT NULL,
  enrollment_date DATE DEFAULT CURRENT_DATE,
  status VARCHAR(20) DEFAULT 'active',
  roll_number INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (student_id) REFERENCES users(id),
  FOREIGN KEY (class_id) REFERENCES classes(id),
  FOREIGN KEY (academic_year_id) REFERENCES academic_years(id),
  UNIQUE(student_id, class_id, academic_year_id),
  INDEX idx_student_year (student_id, academic_year_id),
  INDEX idx_class (class_id)
);
```

### Subjects
```sql
CREATE TABLE subjects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  institution_id UUID NOT NULL,
  name VARCHAR(100) NOT NULL,
  code VARCHAR(50),
  description TEXT,
  is_optional BOOLEAN DEFAULT FALSE,
  is_practical BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (institution_id) REFERENCES institutions(id),
  UNIQUE(institution_id, code)
);

CREATE TABLE subject_classes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subject_id UUID NOT NULL,
  class_id UUID NOT NULL,
  teacher_id UUID NOT NULL,
  periods_per_week INT DEFAULT 5,
  max_marks INT DEFAULT 100,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (subject_id) REFERENCES subjects(id),
  FOREIGN KEY (class_id) REFERENCES classes(id),
  FOREIGN KEY (teacher_id) REFERENCES users(id),
  UNIQUE(subject_id, class_id, teacher_id)
);
```

### Academic Year
```sql
CREATE TABLE academic_years (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  institution_id UUID NOT NULL,
  year VARCHAR(20),
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  is_current BOOLEAN DEFAULT FALSE,
  status VARCHAR(20) DEFAULT 'active',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (institution_id) REFERENCES institutions(id),
  UNIQUE(institution_id, year)
);
```

### Attendance
```sql
CREATE TABLE attendance (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID NOT NULL,
  class_id UUID NOT NULL,
  subject_id UUID,
  attendance_date DATE NOT NULL,
  period INT,
  status VARCHAR(20) NOT NULL,
  marked_by_id UUID NOT NULL,
  remarks TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (student_id) REFERENCES users(id),
  FOREIGN KEY (class_id) REFERENCES classes(id),
  FOREIGN KEY (subject_id) REFERENCES subjects(id),
  FOREIGN KEY (marked_by_id) REFERENCES users(id),
  INDEX idx_student_date (student_id, attendance_date),
  INDEX idx_class_date (class_id, attendance_date)
);
```

### Assignments
```sql
CREATE TABLE assignments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  subject_class_id UUID NOT NULL,
  created_by_id UUID NOT NULL,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  instructions TEXT,
  attachment_urls JSONB,
  assignment_date DATE NOT NULL,
  due_date DATE NOT NULL,
  max_marks INT DEFAULT 10,
  status VARCHAR(20) DEFAULT 'published',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (subject_class_id) REFERENCES subject_classes(id),
  FOREIGN KEY (created_by_id) REFERENCES users(id),
  INDEX idx_class_subject (subject_class_id),
  INDEX idx_due_date (due_date)
);

CREATE TABLE assignment_submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  assignment_id UUID NOT NULL,
  student_id UUID NOT NULL,
  submission_file_urls JSONB NOT NULL,
  submitted_at TIMESTAMP,
  status VARCHAR(20) DEFAULT 'not_submitted',
  marks_obtained INT,
  grade VARCHAR(2),
  feedback TEXT,
  graded_at TIMESTAMP,
  graded_by_id UUID,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (assignment_id) REFERENCES assignments(id),
  FOREIGN KEY (student_id) REFERENCES users(id),
  FOREIGN KEY (graded_by_id) REFERENCES users(id),
  UNIQUE(assignment_id, student_id),
  INDEX idx_assignment (assignment_id),
  INDEX idx_student (student_id)
);
```

### Exams & Results
```sql
CREATE TABLE examinations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  academic_year_id UUID NOT NULL,
  name VARCHAR(255) NOT NULL,
  start_date DATE NOT NULL,
  end_date DATE NOT NULL,
  status VARCHAR(20) DEFAULT 'scheduled',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (academic_year_id) REFERENCES academic_years(id),
  INDEX idx_year_status (academic_year_id, status)
);

CREATE TABLE exam_results (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID NOT NULL,
  exam_id UUID NOT NULL,
  subject_id UUID NOT NULL,
  marks_obtained INT NOT NULL,
  grade VARCHAR(2),
  is_passed BOOLEAN,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (student_id) REFERENCES users(id),
  FOREIGN KEY (exam_id) REFERENCES examinations(id),
  FOREIGN KEY (subject_id) REFERENCES subjects(id),
  INDEX idx_student (student_id)
);
```

### Messages & Announcements
```sql
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sender_id UUID NOT NULL,
  recipient_id UUID,
  group_id UUID,
  content TEXT NOT NULL,
  message_type VARCHAR(50) DEFAULT 'text',
  attachment_urls JSONB,
  is_read BOOLEAN DEFAULT FALSE,
  read_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (sender_id) REFERENCES users(id),
  FOREIGN KEY (recipient_id) REFERENCES users(id),
  INDEX idx_recipient (recipient_id),
  INDEX idx_created_at (created_at)
);

CREATE TABLE announcements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  institution_id UUID NOT NULL,
  created_by_id UUID NOT NULL,
  title VARCHAR(255) NOT NULL,
  content TEXT NOT NULL,
  target_audience JSONB NOT NULL,
  attachment_urls JSONB,
  is_published BOOLEAN DEFAULT TRUE,
  published_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  expiry_date TIMESTAMP,
  priority VARCHAR(20) DEFAULT 'normal',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (institution_id) REFERENCES institutions(id),
  FOREIGN KEY (created_by_id) REFERENCES users(id),
  INDEX idx_published (is_published)
);
```

### Advanced Features
```sql
CREATE TABLE books (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  institution_id UUID NOT NULL,
  title VARCHAR(255) NOT NULL,
  author VARCHAR(255),
  isbn VARCHAR(20),
  publisher VARCHAR(255),
  publication_year INT,
  category VARCHAR(100),
  description TEXT,
  total_copies INT NOT NULL DEFAULT 1,
  available_copies INT NOT NULL DEFAULT 1,
  cover_image_url TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (institution_id) REFERENCES institutions(id),
  INDEX idx_title (title),
  INDEX idx_isbn (isbn)
);

CREATE TABLE health_records (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  student_id UUID NOT NULL,
  blood_group VARCHAR(10),
  allergies TEXT,
  medical_conditions TEXT,
  height_cm DECIMAL(5, 2),
  weight_kg DECIMAL(5, 2),
  last_checkup_date DATE,
  last_checkup_notes TEXT,
  emergency_contact_name VARCHAR(255),
  emergency_contact_phone VARCHAR(20),
  emergency_contact_relation VARCHAR(50),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (student_id) REFERENCES users(id),
  UNIQUE(student_id)
);
```

### Analytics & Audit
```sql
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL,
  entity_type VARCHAR(100),
  entity_id UUID,
  action VARCHAR(50),
  old_values JSONB,
  new_values JSONB,
  ip_address VARCHAR(50),
  user_agent TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id),
  INDEX idx_entity (entity_type, entity_id),
  INDEX idx_user (user_id),
  INDEX idx_created_at (created_at)
);

CREATE TABLE login_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL,
  login_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  logout_time TIMESTAMP,
  ip_address VARCHAR(50),
  device_info TEXT,
  browser_info TEXT,
  session_id VARCHAR(255),
  status VARCHAR(20),
  FOREIGN KEY (user_id) REFERENCES users(id),
  INDEX idx_user (user_id),
  INDEX idx_login_time (login_time)
);
```

---

**Last Updated**: 2026
**Version**: 1.0.0