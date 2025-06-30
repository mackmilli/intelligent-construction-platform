Data Dictionary & Schema
This document defines the structure for all data stored in the Postgres database. It serves as the single source of truth for database table and column definitions.

Table: users
Stores user account information.

Column Name

Data Type

Constraints

Description

id

UUID

PRIMARY KEY

Unique identifier for the user.

email

VARCHAR(255)

UNIQUE, NOT NULL

User's email address for login.

password_hash

VARCHAR(255)

NOT NULL

Hashed user password using bcrypt.

full_name

VARCHAR(255)



User's full name.

created_at

TIMESTAMP

DEFAULT NOW()

Timestamp when the user account was created.

updated_at

TIMESTAMP

DEFAULT NOW()

Timestamp when the user account was last updated.

Table: projects
Stores information about construction projects.

Column Name

Data Type

Constraints

Description

id

UUID

PRIMARY KEY

Unique identifier for the project.

name

VARCHAR(255)

NOT NULL

The official name of the project.

location

VARCHAR(255)



The physical address or location of the project.

owner_id

UUID

FOREIGN KEY (users.id)

The user who owns or created this project entry.

created_at

TIMESTAMP

DEFAULT NOW()

Timestamp when the project was created.

updated_at

TIMESTAMP

DEFAULT NOW()

Timestamp when the project was last updated.

Table: files
Stores metadata for all uploaded files, linking them to projects.

Column Name

Data Type

Constraints

Description

id

UUID

PRIMARY KEY

Unique identifier for the file.

project_id

UUID

FOREIGN KEY (projects.id)

The project this file belongs to.

uploader_id

UUID

FOREIGN KEY (users.id)

The user who uploaded the file.

file_name

VARCHAR(255)

NOT NULL

Original name of the uploaded file.

file_type

VARCHAR(50)



MIME type of the file (e.g., 'application/pdf').

storage_path

VARCHAR(1024)

NOT NULL

The path to the file in MinIO object storage.

file_size

BIGINT



Size of the file in bytes.

upload_status

VARCHAR(50)

DEFAULT 'pending'

Status of the file upload/processing (e.g., 'pending', 'uploaded', 'processing', 'complete', 'error').

created_at

TIMESTAMP

DEFAULT NOW()

Timestamp when the file was uploaded.

updated_at

TIMESTAMP

DEFAULT NOW()

Timestamp when the file record was last updated.

Table: analysis_results
Stores the output from the AI/LLM analysis jobs.

Column Name

Data Type

Constraints

Description

id

UUID

PRIMARY KEY

Unique identifier for the analysis result.

file_id

UUID

FOREIGN KEY (files.id)

The file that was analyzed.

analysis_type

VARCHAR(100)



The type of analysis performed (e.g., 'summary', 'object_detection').

result_data

JSONB

NOT NULL

The structured JSON output from the analysis.

created_at

TIMESTAMP

DEFAULT NOW()

Timestamp when the analysis was completed.

