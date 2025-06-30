Intelligent Construction Analysis Platform: The Complete Master Tool Stack
This document provides a comprehensive list of all software and services required to build the Intelligent Construction Analysis Platform, detailing how each component connects and what credentials are needed.

I. Core Infrastructure & Development Environment
These tools form the foundational layer for building and running the application on a local machine.

Tool

Role in System

Phase

Required Connects

Example of What's Needed

WSL 2

Linux Environment on Windows

1

Installs a Linux distribution (like Ubuntu) within Windows. Docker Desktop integrates directly with it for better performance.

An installed Linux distribution from the Microsoft Store (e.g., "Ubuntu 22.04 LTS").

Docker

Containerization Platform

1

Runs all other services (API, DB, etc.) in isolated containers, allowing them to communicate over a virtual network.

A Dockerfile for each service (API, workers) defining its environment.

Docker Compose

Local Environment Orchestrator

1

Uses a single configuration file (docker-compose.yml) to define, link, and run all the service containers together.

A docker-compose.yml file listing all services and their required environment variables.

II. Backend Services & API
These components make up the server-side "engine" of the platform.

Tool

Role in System

Phase

Required Connects

Example of What's Needed

Python

Primary Backend Language

1

No connection needed; it's the language used to write the API and AI workers.

A requirements.txt file listing all Python library dependencies.

FastAPI

API Framework

1

Runs inside the API container and needs network access to Postgres, Redis, and MinIO containers.

Python code that initializes connections using the URLs/keys from environment variables.

Pydantic

Data Validation

1

Integrated directly into the FastAPI Python code; requires no external connection.

Python classes defining the expected structure of API request data.

Postgres

Primary Database

1

The API and AI Workers need network access to the Postgres container.

A full database connection URL stored as an environment variable. Ex: DB_URL=postgresql://user:password@postgres:5432/construction_db

MinIO

Object Storage

1

The API needs credentials to generate upload links. Workers need credentials to fetch files.

1. Root User/Password for administration. 2. An Access Key & Secret Key for the API to use.

Redis

Job Queue & Cache

1

The API and AI Workers need network access to the Redis container.

A Redis connection URL stored as an environment variable. Ex: REDIS_URL=redis://redis:6379

bcrypt

Password Security

1

A Python library imported directly into the API code; requires no external connection.

Code that calls the bcrypt.hashpw() function to secure user passwords before storing them.

III. AI & Data Processing
These tools are used to build the custom machine learning models and handle heavy data processing.

Tool

Role in System

Phase

Required Connects

Example of What's Needed

Celery

Task Queue Manager

1

Configured within the AI Worker's Python code to connect to the Redis broker URL.

A celery_app.py configuration file specifying the Redis URL as the broker.

PyTorch / TensorFlow

Core ML Frameworks

1

Libraries imported into the AI Worker's Python code. Models require access to data stored in MinIO.

A trained model file (e.g., model.pth or model.h5) that the worker loads into memory.

DVC

AI Dataset Versioning

1

Configured to use MinIO as remote storage. It tracks data pointers stored in Git.

DVC config file pointing to your MinIO bucket and credentials.

FFmpeg

Video Pre-Processing

2

A command-line tool installed inside the Video Worker's Docker container.

A Python script in the worker that runs FFmpeg commands using subprocess.

YOLOv8

Video Object Detection Model

2

A pre-trained model file (yolov8n.pt) that is loaded and fine-tuned using PyTorch/TensorFlow.

The .pt model file and your custom annotated video dataset for fine-tuning.

IV. Large Language Model (LLM) Integration
This is the "expert brain" layer for generating advanced insights and summaries.

Tool

Role in System

Phase

Required Connects

Example of What's Needed

OpenAI API (GPT-4)

LLM for Expert Analysis

2

The LLM Worker makes secure HTTPS requests to the OpenAI API endpoint.

An OpenAI API Key stored securely as an environment variable. Ex: OPENAI_API_KEY="sk-..."

Git

Prompt Management

2

A separate Git repository to store, version, and collaborate on the system's LLM prompts.

A folder structure in a Git repo, e.g., /prompts/summarization/v1.txt.

V. Frontend (User Interface)
These tools are used to build the client-facing web application that users interact with.

Tool

Role in System

Phase

Required Connects

Example of What's Needed

React / Vue.js

Core UI Framework

1

The compiled web application makes HTTPS requests to your backend FastAPI endpoints.

The base URL of your API stored as an environment variable in the UI code. Ex: API_BASE_URL="https://api.yourapp.com"

Uppy.js

Robust File Uploader

1

Configured in the UI to first request a presigned URL from your API, then upload the file directly to that MinIO URL.

JavaScript code that configures Uppy to use the AwsS3 plugin, pointing to your API endpoint.

Figma

UI/UX Design

2

No direct system connection; it's the design tool used to create the visual blueprint for the UI.

A shared Figma project file containing all app screens, components, and design specifications.

OpenSeaDragon

Deep Zoom Viewer

2

A JavaScript library in the UI that is given a URL to a tile source (blueprint images) served from MinIO or your API.

JavaScript code to initialize the viewer and point it to the image assets.

SSE

Real-Time UI Updates

2

The UI opens a persistent connection to a specific SSE endpoint on your FastAPI server.

The URL for the SSE endpoint. Ex: https://api.yourapp.com/v1/stream_status

VI. Workflow Automation & Workspace Integration
These tools connect your platform to external business systems to automate workflows.

Tool

Role in System

Phase

Required Connects

Example of What's Needed

n8n

Workflow Automation Engine

3

Your API makes a call to an n8n webhook URL. N8n then connects to your Postgres DB and external services.

A unique Webhook URL from n8n that your API can trigger upon job completion.

Google Workspace

External Business Suite

3

Connect n8n to your Google account using OAuth 2.0, authorizing access to specific services like Sheets and Docs.

A credentials.json file downloaded from a Google Cloud Platform project that you upload into n8n.

Smartsuite

External Project Management

3

N8n makes secure HTTPS requests to the Smartsuite API endpoint using a specific token.

A Smartsuite Workspace ID and a private API Token for authentication.

VII. Production Deployment & Operations (DevOps)
These tools are required to run, scale, monitor, and secure the application in a live production environment.

Tool

Role in System

Phase

Required Connects

Example of What's Needed

Kubernetes (K8s)

Production Orchestrator

4

Your kubectl command-line tool needs a configuration file to connect to your chosen cloud provider's cluster.

A kubeconfig file containing cluster endpoint, user, and authentication details.

Helm

Kubernetes Package Manager

4

A command-line tool that uses your kubeconfig file to deploy "charts" (packages of apps) to your cluster.

A values.yml file that customizes the chart settings for your application.

Cloud Provider

Hosting Infrastructure

4

Your account for a provider like AWS, Google Cloud, or Azure where your K8s cluster and other resources will live.

A registered account with a valid payment method and an IAM user with administrative privileges.

KEDA

Event-Driven Autoscaling

4

Deployed into your Kubernetes cluster, it needs permission to monitor the Redis queue length and manage pod scaling.

A ScaledObject YAML file defining the scaling rules (e.g., target queue length).

Prometheus

Metrics & Monitoring

4

Deployed into Kubernetes, it "scrapes" metrics data from endpoints exposed by your other services.

Configuration defining which service endpoints to scrape for metrics.

Grafana

Monitoring Dashboards

4

Deployed into Kubernetes, it connects to Prometheus and Loki as data sources.

A username and password to log in. The URLs for your Prometheus and Loki data sources.

Loki

Log Aggregation

4

A small agent (Promtail) is installed on each Kubernetes node to collect logs and send them to the central Loki service.

Configuration for the Promtail agent telling it where to find container logs.

HashiCorp Vault

Secrets Management

4

Your applications, running in Kubernetes, will authenticate with Vault to securely fetch their required secrets.

A Vault Token or an AppRole ID/Secret ID for each application to authenticate.

Trivy

Continuous Security Scan

4

Integrated into your CI/CD pipeline (e.g., GitHub Actions) to scan Docker images before they are pushed.

A workflow .yml file in your GitHub repository that includes a step to run the Trivy scan.

pg_dump

Database Backup Utility

4

A script running on a schedule (e.g., a Kubernetes CronJob) that uses the DB connection URL to create a backup file.

The same DB_URL your application uses, with permissions to read all tables.

VIII. Business & User Engagement
These services support user interaction, analytics, and communication.

Tool

Role in System

Phase

Required Connects

Example of What's Needed

PostHog / Mixpanel

User Behavior Analytics

2

Your frontend UI code will include a small JavaScript snippet that sends tracking data to the service.

A unique Project API Key or Token from the analytics service.

SendGrid

Transactional Email

2

Your backend API makes secure HTTPS requests to the SendGrid API endpoint.

A SendGrid API Key with permissions to send emails.

Help Desk Software

User Support & Docs

4

Typically a standalone service. You would add a link to it from your main application UI.

A username and password for your support team to log into the help desk system (e.g., Zendesk).

Screen Recording

Video Tutorial Creation

4

An offline tool (like Camtasia or Loom) to record your screen. The final video files can be hosted on YouTube or Vimeo.

The software application itself. No system connects are needed.

IX. Code Quality & Standards
These tools enforce code consistency and the feature-based architecture.

Tool

Role in System

Phase

Required Connects

Example of What's Needed

ESLint

Code Linter

1

Integrated into the development environment and CI/CD pipeline to analyze code as it's written and committed.

An .eslintrc.js file at the root of the project defining the ruleset.

eslint-plugin-boundaries

Architectural Enforcement

1

A plugin for ESLint that is configured in the .eslintrc.js file.

The specific boundaries/elements and boundaries/element-types rules added to the ESLint config file.

