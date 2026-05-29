# Database Schema - MentorSys (Real-time Systems Edition)

Entity-Relationship Diagram and database design optimized for real-time systems using Liquibase.

## Real-time Systems Design Considerations

For a real-time mentorship platform, the database schema must support:

1. **High Concurrency**: Multiple users connecting/disconnecting simultaneously
2. **Low Latency**: Fast reads for presence, notifications, session state
3. **Event Sourcing**: Track all state changes for real-time updates
4. **Distributed Systems**: Support horizontal scaling with event synchronization
5. **Change Capture**: Detect updates for WebSocket broadcasts

---

## ER Diagram (Optimized for Real-time)

```
┌──────────────────────────────────────────────────────────────────────────┐
│                        USERS (Core Entity)                               │
├──────────────────────────────────────────────────────────────────────────┤
│ id (PK, BigInt)          │ email (UNIQUE, INDEXED)                       │
│ first_name               │ last_name                                      │
│ password_hash            │ role (INDEXED: MENTOR/MENTEE)                 │
│ active (INDEXED)         │ online_status (INDEXED: for real-time)         │
│ last_activity_at         │ created_at                                     │
│ updated_at               │ version (for optimistic locking)              │
└──────────────────────┬───────────────────────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        │ 1:1          │ 1:1          │ 1:N
        ▼              ▼              ▼
   ┌─────────────┐ ┌─────────────┐ ┌────────────────────────────┐
   │   MENTORS   │ │   MENTEES   │ │   USER_PRESENCE_EVENTS     │
   ├─────────────┤ ├─────────────┤ ├────────────────────────────┤
   │ id (PK)     │ │ id (PK)     │ │ id (PK)                    │
   │ user_id(FK) │ │ user_id(FK) │ │ user_id (FK, INDEXED)      │
   │ title       │ │ level       │ │ status (ONLINE/OFFLINE)    │
   │ bio         │ │ goals       │ │ device_type                │
   │ expertise   │ │ created_at  │ │ ip_address                 │
   │ hourly_rate │ │ updated_at  │ │ created_at (INDEXED)       │
   │ rating      │ │             │ │ session_id (nullable)      │
   │ created_at  │ └─────────────┘ └────────────────────────────┘
   │ updated_at  │
   └────┬────────┘
        │
        │ 1:N (Mentor)
        │
        ├─────────────────────────────────────────────────┐
        │                                                 │
        │ N (Mentor)                                      │ N (Mentee)
        ▼                                                 ▼
   ┌──────────────────────────────────────────────────────────────────────┐
   │                           SESSIONS                                   │
   ├──────────────────────────────────────────────────────────────────────┤
   │ id (PK)                      │ mentor_id (FK, INDEXED)               │
   │ mentee_id (FK, INDEXED)      │ title                                 │
   │ description                  │ scheduled_at (INDEXED)                │
   │ duration (minutes)           │ session_type                          │
   │ status (INDEXED: for queries) │ meeting_link                         │
   │ notes                        │ rating                                │
   │ started_at                   │ completed_at                          │
   │ cancelled_at                 │ cancellation_reason                   │
   │ created_at (INDEXED)         │ updated_at                            │
   │ version (optimistic locking) │                                       │
   └──────────────┬───────────────────────────────────────────────────────┘
                  │
                  │ 1:N
                  ▼
   ┌──────────────────────────────────────────┐
   │        SESSION_MESSAGES                  │
   ├──────────────────────────────────────────┤
   │ id (PK)                                  │
   │ session_id (FK, INDEXED)                 │
   │ sender_id (FK)                           │
   │ content                                  │
   │ message_type (TEXT/SYSTEM/FILE)          │
   │ attachment_url                           │
   │ created_at (INDEXED)                     │
   │ is_edited                                │
   └──────────────────────────────────────────┘
                  │
                  │
   ┌──────────────────────────────────────────┐
   │      SESSION_STATE_CHANGES                │ ◄─── Event Sourcing
   ├──────────────────────────────────────────┤
   │ id (PK)                                  │
   │ session_id (FK, INDEXED)                 │
   │ changed_by (FK)                          │
   │ old_state (JSON)                         │
   │ new_state (JSON)                         │
   │ change_type                              │
   │ created_at (INDEXED)                     │
   └──────────────────────────────────────────┘

        ┌─────────────────────────────────────────────────┐
        │                                                 │
        ▼                                                 ▼
   ┌──────────────────────┐                    ┌──────────────────────────┐
   │  NOTIFICATIONS       │                    │   MESSAGE_DELIVERY_LOG   │
   ├──────────────────────┤                    ├──────────────────────────┤
   │ id (PK)              │                    │ id (PK)                  │
   │ user_id (FK)         │                    │ message_id (FK)          │
   │ type (INDEXED)       │                    │ recipient_id (FK)        │
   │ title                │                    │ delivery_status          │
   │ message              │                    │ delivered_at             │
   │ read (INDEXED)       │                    │ read_at                  │
   │ data (JSON)          │                    │ created_at               │
   │ created_at (INDEXED) │                    └──────────────────────────┘
   │ read_at              │
   └──────────────────────┘
```

---

## Table Definitions with Real-time Optimizations

### 1. USERS Table

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(20) NOT NULL,  -- MENTOR, MENTEE
    active BOOLEAN DEFAULT TRUE,
    online_status VARCHAR(50) DEFAULT 'OFFLINE',  -- For real-time presence
    last_activity_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    version BIGINT DEFAULT 0,  -- Optimistic locking for concurrent updates
    
    CONSTRAINT user_role_check CHECK (role IN ('MENTOR', 'MENTEE')),
    CONSTRAINT user_status_check CHECK (online_status IN ('ONLINE', 'OFFLINE', 'AWAY', 'BUSY'))
);

-- Indexes for real-time queries
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
CREATE INDEX idx_users_active ON users(active);
CREATE INDEX idx_users_online_status ON users(online_status);  -- For presence tracking
CREATE INDEX idx_users_last_activity ON users(last_activity_at DESC);
```

**Real-time Fields:**
- `online_status`: Tracks user presence (broadcast via WebSocket)
- `last_activity_at`: Detect idle users, heartbeat tracking
- `version`: Prevent concurrent update conflicts in distributed systems

---

### 2. USER_PRESENCE_EVENTS Table (Event Sourcing)

```sql
CREATE TABLE user_presence_events (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL,
    status VARCHAR(50) NOT NULL,
    device_type VARCHAR(50),
    ip_address INET,
    session_id BIGINT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT presence_status_check CHECK (status IN ('ONLINE', 'OFFLINE', 'AWAY', 'IDLE'))
);

-- For real-time queries and retention
CREATE INDEX idx_presence_user_created ON user_presence_events(user_id, created_at DESC);
CREATE INDEX idx_presence_created ON user_presence_events(created_at DESC);
CREATE INDEX idx_presence_status ON user_presence_events(status);

-- Partition by date for performance (optional for large systems)
-- PARTITION BY RANGE (YEAR(created_at), MONTH(created_at))
```

**Purpose:** 
- Track user presence changes for real-time status updates
- Event sourcing: Complete audit trail of presence changes
- Detect disconnections and cleanup stale connections

---

### 3. MENTORS Table

```sql
CREATE TABLE mentors (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL UNIQUE,
    title VARCHAR(255) NOT NULL,
    bio TEXT,
    expertise TEXT[] NOT NULL,  -- Array of skills (PostgreSQL)
    hourly_rate NUMERIC(10, 2),
    rating NUMERIC(3, 2) DEFAULT 0,
    sessions_completed INT DEFAULT 0,
    response_time_avg INT,  -- Average response time in seconds
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    version BIGINT DEFAULT 0,
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE INDEX idx_mentors_user_id ON mentors(user_id);
CREATE INDEX idx_mentors_rating ON mentors(rating DESC);
CREATE INDEX idx_mentors_expertise ON mentors USING GIN(expertise);  -- For skill search
CREATE INDEX idx_mentors_response_time ON mentors(response_time_avg ASC);
```

---

### 4. MENTEES Table

```sql
CREATE TABLE mentees (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL UNIQUE,
    experience_level VARCHAR(50),  -- BEGINNER, INTERMEDIATE, ADVANCED
    goals TEXT,
    learning_pace VARCHAR(50),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT mentee_level_check CHECK (experience_level IN ('BEGINNER', 'INTERMEDIATE', 'ADVANCED')),
    CONSTRAINT mentee_pace_check CHECK (learning_pace IN ('SLOW', 'MODERATE', 'FAST'))
);

CREATE INDEX idx_mentees_user_id ON mentees(user_id);
```

---

### 5. SESSIONS Table (Critical for Real-time)

```sql
CREATE TABLE sessions (
    id BIGSERIAL PRIMARY KEY,
    mentor_id BIGINT NOT NULL,
    mentee_id BIGINT NOT NULL,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    scheduled_at TIMESTAMP NOT NULL,
    duration_minutes INT NOT NULL,
    session_type VARCHAR(50) NOT NULL,  -- MENTORING, TRAINING, COACHING
    status VARCHAR(50) NOT NULL,  -- SCHEDULED, STARTED, IN_PROGRESS, COMPLETED, CANCELLED
    meeting_link VARCHAR(500),
    meeting_status VARCHAR(50),  -- For real-time meeting state
    notes TEXT,
    rating INT,
    started_at TIMESTAMP,
    completed_at TIMESTAMP,
    cancelled_at TIMESTAMP,
    cancellation_reason TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    version BIGINT DEFAULT 0,  -- Optimistic locking
    
    FOREIGN KEY (mentor_id) REFERENCES users(id) ON DELETE RESTRICT,
    FOREIGN KEY (mentee_id) REFERENCES users(id) ON DELETE RESTRICT,
    CONSTRAINT session_status_check CHECK (status IN 
        ('SCHEDULED', 'STARTED', 'IN_PROGRESS', 'COMPLETED', 'CANCELLED')),
    CONSTRAINT session_type_check CHECK (session_type IN 
        ('MENTORING', 'TRAINING', 'COACHING'))
);

-- Critical indexes for real-time queries
CREATE INDEX idx_sessions_mentor_id ON sessions(mentor_id);
CREATE INDEX idx_sessions_mentee_id ON sessions(mentee_id);
CREATE INDEX idx_sessions_status ON sessions(status);  -- For active sessions
CREATE INDEX idx_sessions_scheduled ON sessions(scheduled_at DESC);
CREATE INDEX idx_sessions_created ON sessions(created_at DESC);
CREATE INDEX idx_sessions_mentor_status ON sessions(mentor_id, status);  -- Composite
CREATE INDEX idx_sessions_mentee_status ON sessions(mentee_id, status);  -- Composite

-- For upcoming sessions within 24 hours
CREATE INDEX idx_sessions_upcoming ON sessions(scheduled_at) 
    WHERE status = 'SCHEDULED' AND scheduled_at > CURRENT_TIMESTAMP;
```

---

### 6. SESSION_STATE_CHANGES Table (Event Sourcing)

```sql
CREATE TABLE session_state_changes (
    id BIGSERIAL PRIMARY KEY,
    session_id BIGINT NOT NULL,
    changed_by BIGINT NOT NULL,
    old_state JSONB,  -- Use JSONB for better query performance
    new_state JSONB,
    change_type VARCHAR(50),  -- STATUS_CHANGE, PARTICIPANT_JOINED, MEETING_STARTED, etc.
    description TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (session_id) REFERENCES sessions(id) ON DELETE CASCADE,
    FOREIGN KEY (changed_by) REFERENCES users(id) ON DELETE SET NULL
);

-- For real-time event streaming
CREATE INDEX idx_state_changes_session ON session_state_changes(session_id, created_at DESC);
CREATE INDEX idx_state_changes_created ON session_state_changes(created_at DESC);
CREATE INDEX idx_state_changes_type ON session_state_changes(change_type);

-- JSONB indexing for complex queries
CREATE INDEX idx_state_changes_new_state ON session_state_changes USING GIN(new_state);
```

**Purpose:**
- Complete audit trail of session state transitions
- Enables event replay and debugging
- Powers real-time updates via WebSocket

---

### 7. SESSION_MESSAGES Table

```sql
CREATE TABLE session_messages (
    id BIGSERIAL PRIMARY KEY,
    session_id BIGINT NOT NULL,
    sender_id BIGINT NOT NULL,
    content TEXT NOT NULL,
    message_type VARCHAR(50) DEFAULT 'TEXT',  -- TEXT, SYSTEM, FILE, CODE
    attachment_url VARCHAR(500),
    is_edited BOOLEAN DEFAULT FALSE,
    edited_at TIMESTAMP,
    deleted_at TIMESTAMP,  -- Soft delete
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (session_id) REFERENCES sessions(id) ON DELETE CASCADE,
    FOREIGN KEY (sender_id) REFERENCES users(id) ON DELETE SET NULL
);

-- For real-time message streaming
CREATE INDEX idx_messages_session_created ON session_messages(session_id, created_at ASC);
CREATE INDEX idx_messages_sender ON session_messages(sender_id);
CREATE INDEX idx_messages_created ON session_messages(created_at DESC);

-- For pagination and lazy loading
CREATE INDEX idx_messages_session_id_desc ON session_messages(session_id, id DESC);
```

---

### 8. NOTIFICATIONS Table (Real-time Critical)

```sql
CREATE TABLE notifications (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL,
    type VARCHAR(100) NOT NULL,  -- SESSION_SCHEDULED, SESSION_REMINDER, MESSAGE_RECEIVED, etc.
    title VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    "read" BOOLEAN DEFAULT FALSE,
    read_at TIMESTAMP,
    data JSONB,  -- Contains references: sessionId, mentorId, messageId, etc.
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP,  -- Auto-delete old notifications
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    CONSTRAINT notification_type_check CHECK (type IN (
        'SESSION_SCHEDULED', 'SESSION_REMINDER', 'MESSAGE_RECEIVED',
        'SESSION_COMPLETED', 'SESSION_CANCELLED', 'MENTOR_RATING'
    ))
);

-- Critical for real-time notification delivery
CREATE INDEX idx_notifications_user_read ON notifications(user_id, "read");
CREATE INDEX idx_notifications_user_created ON notifications(user_id, created_at DESC);
CREATE INDEX idx_notifications_created ON notifications(created_at DESC);
CREATE INDEX idx_notifications_expires ON notifications(expires_at) 
    WHERE expires_at IS NOT NULL;
```

---

### 9. MESSAGE_DELIVERY_LOG Table (Reliability)

```sql
CREATE TABLE message_delivery_log (
    id BIGSERIAL PRIMARY KEY,
    notification_id BIGINT NOT NULL,
    recipient_id BIGINT NOT NULL,
    delivery_status VARCHAR(50) NOT NULL,  -- PENDING, DELIVERED, FAILED, BOUNCED
    delivery_channel VARCHAR(50),  -- WEBSOCKET, EMAIL, PUSH
    attempt_count INT DEFAULT 0,
    last_attempt_at TIMESTAMP,
    delivered_at TIMESTAMP,
    read_at TIMESTAMP,
    error_message TEXT,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (notification_id) REFERENCES notifications(id) ON DELETE CASCADE,
    FOREIGN KEY (recipient_id) REFERENCES users(id) ON DELETE CASCADE
);

-- For real-time delivery tracking and retries
CREATE INDEX idx_delivery_status ON message_delivery_log(delivery_status);
CREATE INDEX idx_delivery_recipient ON message_delivery_log(recipient_id, delivery_status);
CREATE INDEX idx_delivery_created ON message_delivery_log(created_at DESC);
```

---

## Real-time Optimization Strategies

### 1. Connection Pooling Configuration
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 20000
      idle-timeout: 300000
      max-lifetime: 1200000
```

### 2. Query Optimization for Real-time

```sql
-- View for active sessions (frequently accessed)
CREATE VIEW active_sessions AS
SELECT 
    s.id, s.mentor_id, s.mentee_id, s.status,
    m.first_name as mentor_first_name,
    me.first_name as mentee_first_name,
    s.scheduled_at, s.meeting_link
FROM sessions s
JOIN users m ON s.mentor_id = m.id
JOIN users me ON s.mentee_id = me.id
WHERE s.status IN ('STARTED', 'IN_PROGRESS');

-- View for user online status
CREATE VIEW user_status_summary AS
SELECT 
    u.id, u.email, u.online_status,
    COUNT(DISTINCT CASE WHEN s.status IN ('STARTED', 'IN_PROGRESS') THEN s.id END) as active_sessions,
    MAX(pe.created_at) as last_presence_update
FROM users u
LEFT JOIN user_presence_events pe ON u.id = pe.user_id
LEFT JOIN sessions s ON (u.id = s.mentor_id OR u.id = s.mentee_id)
GROUP BY u.id, u.email, u.online_status;
```

### 3. Partitioning Strategy

For high-volume systems, consider partitioning:

```sql
-- Partition notifications by date
ALTER TABLE notifications 
PARTITION BY RANGE (YEAR(created_at), MONTH(created_at));

-- Partition session_messages by session_id range
ALTER TABLE session_messages 
PARTITION BY HASH (session_id) PARTITIONS 16;
```

---

## Liquibase Integration

Liquibase will be used for:
1. **Version Control**: Track all schema changes
2. **Rollback Support**: Easily revert changes
3. **Multi-environment**: Different configs for dev/test/prod
4. **Team Collaboration**: Prevent conflicts in schema evolution
5. **Audit Trail**: Complete history of migrations

---

## Performance Monitoring Queries

```sql
-- Find slow queries in real-time system
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Check index usage
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Monitor active connections
SELECT datname, usename, state, count(*)
FROM pg_stat_activity
GROUP BY datname, usename, state;
```

---

**Next:** Move to Liquibase configuration for schema versioning.
