# Linux DevOps Automated Deployment Platform

A hands-on DevOps project demonstrating automated web application deployment using Jenkins, Docker, Git, and Shell.

## Project Overview

This project builds and deploys a simple web application through a Jenkins Pipeline. It includes Docker image versioning, deployment health checks, and rollback logic to restore the previous container when a deployment fails.

## Technology Stack

- Linux (Rocky Linux 9)
- Jenkins Pipeline
- Git and GitHub
- Docker
- Shell scripting
- curl for HTTP health checks

## Project Structure

```text
.
├── app/
│   └── index.html
├── Dockerfile
├── Jenkinsfile
├── Jenkinsfile.rollback-test
└── README.md
```

## Deployment Workflow

1. Jenkins checks out the project from GitHub.
2. The pipeline verifies the required project files.
3. Docker builds a versioned image using the Jenkins build number.
4. The existing container is preserved as a rollback container.
5. A new container is started.
6. The pipeline checks the application endpoint using curl.
7. If the health check passes, the new container becomes the active container and the old rollback container is removed.
8. If deployment or health checks fail, the pipeline attempts to restore the previous container.

## Jenkins Jobs

### Production Deployment

- Pipeline file: `Jenkinsfile`
- Container: `devops-web-ci`
- Host port: `8080`
- Container port: `8080`

The production pipeline uses a versioned Docker image and checks the application endpoint before completing deployment.

### Rollback Test

- Pipeline file: `Jenkinsfile.rollback-test`
- Container: `devops-web-rollback-test`
- Host port: `18082`

The rollback test intentionally requests a nonexistent URL to trigger a failed health check. It then removes the failed container, restores the previous container, and checks whether the previous version is healthy.

## Validation Results

The rollback test was run successfully:

- The new version failed its intentional health check.
- The failed container was removed.
- The previous container was restored.
- The rollback health check completed successfully.
- Jenkins finished with `SUCCESS`.
- A separate HTTP check returned `200 OK`.

## Notes

- The rollback test is intentionally isolated from the production container and uses a separate host port.
- The production pipeline and rollback test pipeline have different completion-status behavior. In the production pipeline, a failed deployment health check exits with status `1` even if the previous container is restored.
- This project is a learning environment. Review deployment scripts and recovery behavior before using them in production.

## Author's Learning Goals

- Practice Linux administration and Shell scripting.
- Build CI/CD pipelines with Jenkins.
- Automate container builds and deployments with Docker.
- Validate deployments with HTTP health checks.
- Test rollback and service recovery behavior.
