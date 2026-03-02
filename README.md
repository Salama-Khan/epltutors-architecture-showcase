# EPL Tutors: AI-Powered EdTech Assessment Platform

**Live Application:** [epltutors.com](https://epltutors.com)  
**Guest Access:** Email: `employer_demo@epltutors.com` | Password: `HireMe2026!`

*(Note: The source code for this platform is held in a private repository as it is a live, revenue-generating SaaS business. This repository serves as a technical architecture and system design showcase).*

## 📌 Project Overview
I built and deployed a full-stack, cloud-hosted educational platform designed to automate the grading of UK GCSE Science papers using deterministic routing paired with Generative AI. The system dynamically analyses student weaknesses, generates custom JSON-structured feedback, and serves asynchronous video curriculum.

## 🛠 The Tech Stack
To ensure high availability and secure data handling, I architected the platform using a modern cloud-native stack:

* **Application Logic (SWE):** Python 3.12, Django 5, Django REST Framework
* **Data Engineering:** PostgreSQL (RDS), strictly typed JSON pipelines, OpenAI Async API
* **Infrastructure & DevOps:** AWS Elastic Beanstalk (EC2), AWS S3, Gunicorn, Whitenoise
* **Version Control & CI/CD:** Git, GitHub, EB CLI deployments

## 🏗 System Architecture
*(Insert a screenshot here of a clean AWS Architecture Diagram showing your Django App talking to RDS, S3, and the OpenAI API)*

## 🚀 Core Engineering Features

### 1. Automated AI Grading Engine (Data/SWE)
* Integrated the OpenAI Async API to process unstructured student answers against strict, deterministic JSON mark schemes.
* Engineered the prompt routing to prevent AI hallucination, ensuring 100% compliance with UK exam board standards.
* Designed a relational PostgreSQL schema to track micro-skill mastery, allowing the platform to dynamically route students to specific curriculum videos based on their unique weaknesses.

### 2. Scalable Cloud Infrastructure (DevOps)
* Provisioned and deployed the application to AWS Elastic Beanstalk, configuring the underlying EC2 instances and load balancers for production traffic.
* Configured AWS S3 for secure static and media asset delivery, writing custom IAM bucket policies and managing Object Ownership ACLs to protect premium content.

### 3. Secure Production Data Management (Data/DevOps)
* Implemented strict environment variable isolation. Managed secure injection of database credentials and API keys via the EB CLI and secure SSH sessions, completely abstracting secrets from the codebase.
* Engineered robust data migration pipelines to safely transfer local testing data (`.json` dumps) into the production PostgreSQL RDS instance while handling unique constraint violations and schema synchronisation.

## 🐛 Technical Challenges & Solutions

* **The Problem:** During live deployment, loading local database dumps into the AWS PostgreSQL instance triggered severe `IntegrityError` unique constraint violations and missing column `ProgrammingError` crashes.
* **My Solution:** I executed a secure SSH tunnel into the live Elastic Beanstalk EC2 instance, manually injected the masked environment variables into the bash session, flushed the corrupted schema via the Django shell, and rebuilt the production tables from the ground up before successfully passing the serialized JSON payload.

## 📸 Platform Previews
*(Add 2-3 high-quality screenshots here. Show the student dashboard, the AI feedback screen, and maybe a snippet of your S3/AWS console).*
