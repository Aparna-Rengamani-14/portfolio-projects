## CI/CD Pipeline for Java (Maven) Application using Jenkins

This project demonstrates a **fully automated CI/CD pipeline** built using **Jenkins** for a Java (Maven) application.

The pipeline covers **build, testing, code quality analysis, artifact management, containerization, and notifications**.

This project is designed to showcase **real-world DevOps CI/CD pipeline design** — the Jenkinsfile itself is the

deliverable, built to reflect patterns used in production environments rather than a toy example.

---

## Architecture

![Pipeline Architecture](architecture/pipeline-architecture.png)

The pipeline is structured into four logical stages:

1. **Preparation** — pipeline trigger, tool setup (JDK, Maven), environment variables, and pipeline options

   (build retention, timeout, colored output)

2. **Build & Test** — checkout, build, and artifact archiving, followed by parallel execution of unit tests,

   integration tests, Checkstyle analysis, and SonarQube analysis

3. **Quality Gate & Publish** — enforces a SonarQube quality gate (aborts the pipeline on failure) before

   publishing versioned artifacts to Nexus Repository

4. **Conditional Docker Build & Notifications** — builds and pushes a Docker image only if a Dockerfile is

   present, then sends success/failure email notifications and cleans the workspace

---

## Project Overview

The pipeline automates the following workflow:

1. Fetches source code from Git

2. Builds the Java application using Maven

3. Runs unit and integration tests

4. Performs static code analysis (Checkstyle & SonarQube)

5. Enforces a SonarQube quality gate — the pipeline **aborts** if the gate fails

6. Publishes versioned artifacts to Nexus Repository

7. Builds and pushes a Docker image (if a Dockerfile is present)

8. Sends success/failure email notifications

9. Cleans the Jenkins workspace

---

## Design Decisions

- **Parallel test/quality stages**: Unit tests, integration tests, Checkstyle, and SonarQube analysis run in

  parallel rather than sequentially, reducing pipeline runtime.

- **Quality gate before publish**: Artifacts are only published to Nexus if the SonarQube quality gate passes,

  preventing low-quality code from reaching the artifact repository.

- **Conditional Docker stage**: The Docker build/push stage only runs `when { expression { fileExists('Dockerfile') } }`,

  so the same pipeline works for projects with or without containerization — no separate pipeline needed.

- **Build safeguards**: `buildDiscarder` limits retained builds, and a 30-minute `timeout` prevents a hung

  build from blocking the Jenkins agent indefinitely.

---

## Tools & Technologies Used

| Category | Tools |

|-------|------|

| CI/CD | Jenkins |

| Build Tool | Maven |

| Code Quality | Checkstyle, SonarQube |

| Artifact Repository | Nexus Repository Manager |

| Containerization | Docker |

| Notifications | Email |

| Version Control | Git |

---

## Repository Structure

```text

.

├── Jenkinsfile           # Declarative Jenkins CI/CD pipeline

├── architecture/         # Pipeline architecture diagram

└── README.md             # Project documentation

```

**Note:** This repository contains the pipeline definition and architecture documentation only — it's a

CI/CD pipeline showcase rather than a full runnable application. The Jenkinsfile references a `pom.xml`,

`src/`, and an optional `Dockerfile`, which would be part of the Java application repo this pipeline is

designed to run against.

---

## What I Learned

Building this pipeline helped me understand how quality gates, artifact versioning, and conditional stages

fit together in a real CI/CD workflow — particularly how to structure a pipeline so it fails fast and safely

(aborting on a failed quality gate) rather than silently pushing a bad build forward.
