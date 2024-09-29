
# Project2-Compose: Docker Compose Configuration for Jenkins CI/CD

## Overview

This repository contains the Docker Compose configuration required to set up a Jenkins server integrated with Docker-in-Docker (DinD). This setup is used to create a complete CI/CD environment that automates testing, building, and deploying a Node.js application.

## Services

The `docker-compose.yml` defines two main services:

1. **DinD (Docker-in-Docker) Service**: 
   - Allows Jenkins to communicate directly with Docker to manage containers.
   - Uses a privileged mode to allow Docker operations inside the container.
   - Sets up a secure environment using certificates.

2. **Jenkins Service**:
   - The Jenkins service uses the official Jenkins Docker image.
   - Jenkins is configured to interact with the Docker service to build and push Docker images.
   - The Jenkins server runs in a container and includes support for connecting to the Docker daemon.

### Docker Compose Configuration

```yaml
version: '3'

services:
  dind:
    image: docker:dind
    privileged: true
    environment:
      - DOCKER_TLS_CERTDIR=/certs
    volumes:
      - docker-certs-ca:/certs/ca
      - docker-certs-client:/certs/client
      - jenkins-data:/var/jenkins_home
    networks:
      - jenkins

  jenkins:
    image: jenkins/jenkins:lts-jdk11
    user: root
    environment:
      - DOCKER_HOST=tcp://dind:2376
      - DOCKER_CERT_PATH=/certs/client
      - DOCKER_TLS_VERIFY=1
    volumes:
      - docker-certs-client:/certs/client:ro
      - jenkins-data:/var/jenkins_home
      - /usr/bin/docker:/usr/bin/docker
    ports:
      - "8080:8080"
      - "50000:50000"
    networks:
      - jenkins

volumes:
  docker-certs-ca:
  docker-certs-client:
  jenkins-data:

networks:
  jenkins:
```

### Setup Instructions

1. Clone the repository to your local machine.
2. Ensure Docker and Docker Compose are installed on your system.
3. Run the Docker Compose configuration using:
    ```bash
    docker-compose up -d
    ```
4. Access Jenkins at `http://localhost:8080`. The initial admin password can be found at `/var/jenkins_home/secrets/initialAdminPassword`.

### Usage

- Once Jenkins is running, you can configure it to pull the Jenkinsfile from your application repository (`aws-elastic-beanstalk-express-js-sample`) and run the CI/CD pipeline.

### License

This project is licensed under the MIT License.


---


# AWS Elastic Beanstalk Node.js Sample Application

## Overview

This repository contains a sample Node.js application designed for AWS Elastic Beanstalk deployment, integrated with a Jenkins CI/CD pipeline. The Jenkins pipeline automates the testing, building, and deployment process of the application using Docker.

## Repository Structure

- **Application Code**: The main Node.js application files are located at the root level.
- **Jenkinsfile**: This file contains the CI/CD pipeline configuration, including build, test, security scan, and Docker image deployment stages.

## CI/CD Pipeline Configuration

The pipeline is defined in the `Jenkinsfile` and includes the following stages:

1. **Install Dependencies**:
    - Installs the Node.js dependencies using `npm install`.

2. **Test**:
    - Runs any tests defined in the application. Currently, a placeholder test is provided.

3. **Security Scan**:
    - Uses [Snyk](https://snyk.io) to scan for vulnerabilities. The pipeline will halt if critical vulnerabilities are detected.
    - Note: Due to platform constraints, the Snyk CLI may run in a degraded version. Follow the [documentation](https://docs.snyk.io) to resolve any issues.

4. **Build Docker Image**:
    - Builds a Docker image for the Node.js application.

5. **Push Docker Image**:
    - (Optional) Pushes the Docker image to a container registry (e.g., Docker Hub).

## Running the Application Locally

To run the application locally:

1. Install dependencies:
    ```bash
    npm install
    ```
2. Start the application:
    ```bash
    npm start
    ```
3. Access the application at `http://localhost:3000`.

## Jenkins Pipeline Setup

To set up the Jenkins pipeline:

1. **Pipeline Setup**:
   - Create a new pipeline job in Jenkins and use "Pipeline script from SCM".
   - Provide the URL to this repository, so Jenkins pulls the `Jenkinsfile`.

2. **Running the Pipeline**:
   - Ensure that the Docker and Jenkins environments are set up properly (using `Project2-Compose`).
   - Run the pipeline and verify all stages.

## Logging and Monitoring

- Jenkins logs all build output, including successful builds and failures.
- Snyk reports can be found in the logs, highlighting potential vulnerabilities.

## License

This project is licensed under the MIT License.

### Notes

- Ensure you update the credentials (`docker-hub-credentials`) in Jenkins to successfully push Docker images.
- Additional tests can be added to enhance the reliability of the application.
