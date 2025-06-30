System Architecture & Business Logic
This document describes the high-level architecture of the platform and the core business logic that governs its operations.

I. System Architecture Overview
The platform uses a containerized, microservices-style architecture orchestrated by Docker Compose for local development and Kubernetes for production. This separates concerns and allows for independent scaling of components.

Core Components:
Frontend (React/Vue.js): A Single-Page Application (SPA) that the user interacts with. It is responsible for all UI rendering and user input.

Backend API (FastAPI): The central brain of the application. It handles user authentication, business logic, and communication between all other services.

Database (Postgres): The primary relational database for storing structured data like user accounts, project details, and file metadata.

Object Storage (MinIO): An S3-compatible storage service for holding all large, unstructured files like videos, photos, and blueprints.

Cache & Job Broker (Redis): Used as a message broker for Celery to manage background tasks and for temporary caching of frequently accessed data.

AI Workers (Celery & PyTorch/TensorFlow): Background workers that consume tasks from the Redis queue. They perform the heavy lifting of AI analysis (video processing, LLM summarization) without blocking the API.

II. Core Business Logic: The File Analysis Workflow
This is the primary workflow of the platform. It details the step-by-step process from file upload to report generation.

Trigger: A user uploads one or more files through the frontend.

Presigned URL Request (Frontend -> API):

The frontend (using Uppy.js) makes a request to the API for a secure, temporary "presigned URL" for each file it needs to upload.

Logic: This prevents the need to route large files through our API server. It's a direct, secure upload from the user's browser to the object storage.

Direct Upload (User's Browser -> MinIO):

The API returns the presigned URLs to the frontend.

The frontend then uploads the file data directly to MinIO using those URLs.

Upload Confirmation (MinIO -> API):

Upon successful upload, MinIO can be configured to send a webhook notification to a specific API endpoint.

Logic: This confirms the file is safely stored. The API then updates the file's record in the Postgres database from status: 'pending' to status: 'uploaded'.

Create Analysis Job (API -> Redis):

With the file confirmed, the API creates a new analysis job.

It places a message on the Redis queue. This message contains the file_id and the type of analysis required.

The API updates the file's status in Postgres to status: 'processing'.

Job Consumption (Redis -> AI Worker):

A Celery AI Worker, which is constantly listening to the Redis queue, picks up the job message.

File Processing (AI Worker -> MinIO & External APIs):

The worker fetches the file from MinIO using the file_id.

It performs the required analysis (e.g., runs a YOLOv8 model on a video, sends text to the OpenAI API).

Save Results (AI Worker -> Postgres):

Once the analysis is complete, the worker saves the structured results (e.g., a JSON object) into the analysis_results table in the Postgres database.

Final Status Update (AI Worker -> Postgres):

The worker makes a final call to the API (or directly to the database) to update the file's status to status: 'complete'.

Real-Time Notification (API -> Frontend):

The status update can trigger a Server-Sent Event (SSE) from the API to the user's frontend, updating the UI in real-time to show the job is done without needing a page refresh.