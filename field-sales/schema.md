# Schema — Field Sales

## Tables

### users
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | gen_random_uuid() |
| name | VARCHAR(255) | NOT NULL |
| email | VARCHAR(255) | UNIQUE, NOT NULL |
| password_hash | VARCHAR(255) | bcrypt |
| role | VARCHAR(50) | admin / manager / salesperson |
| phone | VARCHAR(50) | |
| is_active | BOOLEAN | default true |
| created_at | TIMESTAMP | |
| updated_at | TIMESTAMP | |

### customers
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| name | VARCHAR(255) | NOT NULL |
| company | VARCHAR(255) | |
| email | VARCHAR(255) | |
| phone | VARCHAR(50) | |
| address | TEXT | |
| latitude | DOUBLE PRECISION | for map |
| longitude | DOUBLE PRECISION | for map |
| status | VARCHAR(50) | lead / prospect / customer |
| assigned_to | UUID FK → users | |
| notes | TEXT | |
| created_at | TIMESTAMP | |
| updated_at | TIMESTAMP | |

### visits
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| salesperson_id | UUID FK → users | NOT NULL |
| customer_id | UUID FK → customers | NOT NULL |
| check_in_at | TIMESTAMP | GPS timestamp |
| check_out_at | TIMESTAMP | GPS timestamp |
| check_in_lat | DOUBLE PRECISION | |
| check_in_lng | DOUBLE PRECISION | |
| check_out_lat | DOUBLE PRECISION | |
| check_out_lng | DOUBLE PRECISION | |
| notes | TEXT | |
| photos | TEXT[] | MinIO keys |
| status | VARCHAR(50) | planned / checked_in / completed / cancelled |
| created_at | TIMESTAMP | |

### tasks
| Column | Type | Notes |
|--------|------|-------|
| id | UUID PK | |
| title | VARCHAR(255) | NOT NULL |
| description | TEXT | |
| assigned_to | UUID FK → users | salesperson |
| assigned_by | UUID FK → users | manager/admin |
| customer_id | UUID FK → customers | optional |
| due_date | DATE | |
| priority | VARCHAR(50) | low / medium / high / urgent |
| status | VARCHAR(50) | pending / in_progress / completed / cancelled |
| completed_at | TIMESTAMP | |
| created_at | TIMESTAMP | |
| updated_at | TIMESTAMP | |
