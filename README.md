### Streamlining Full Stack Development with Docker, Docker-Compose, Docker Hub, and GitHub Actions

## Introduction

In today's fast-paced software development landscape, Continuous Integration and Continuous Deployment (CI/CD) 
practices are essential for maintaining high-quality code and delivering features to users quickly and efficiently. 
Docker, Docker-Compose, Docker Hub, and GitHub Actions are powerful tools that streamline the development process and automate CI/CD workflows.

In this blog post, we'll explore the process of Dockerizing a full-stack web application consisting of a frontend React application and a backend Django application. We'll cover everything from setting up the project structure to deploying the containerized application using Docker Hub and GitHub Actions for Continuous Integration/Continuous Deployment (CI/CD).

***What We'll Cover:***

 1. Basics: Docker, Docker-Compose, Docker-Hub and Github actions
 2. Project Structure: We'll begin by examining the project structure of both the frontend and backend applications. 
    This will include understanding where to place Dockerfiles, Workflow file, and other necessary configuration 
    files.
 3. Dockerizing Applications: We'll delve into Dockerizing each component of our web application, including creating 
   Dockerfiles with appropriate configurations for the React frontend and Django backend.

 4. Continuous Integration/Continuous Deployment (CI/CD): We'll set up GitHub Actions workflow files to automate the build and deployment process. This will involve configuring workflows to build Docker images and push them to Docker Hub whenever changes are made to the codebase.

5. Deployment with Docker Compose: Finally, we'll deploy our containerized applications on a virtual machine using Docker Compose. We'll see how Docker Compose simplifies the process of managing multi-container Docker applications and orchestrating their deployment.

6. Real-World Use Case: To demonstrate the concepts discussed in this blog post, we'll consider a real-world use case where we have developed a full-stack web application. Our frontend is built using React.js, providing an interactive user interface, while the backend is powered by Django, offering robust server-side functionality.

The project structure comprises directories for both the frontend and backend applications, each containing source code files, configuration files, and Docker-related files. We'll walk through the steps of Dockerizing each component of the application and integrating Docker Hub and GitHub Actions for automated image builds and deployments.

### Section 1: Docker Basics

Docker is a popular platform for developing, shipping, and running applications in containers. Containers encapsulate an application and its dependencies, allowing it to run consistently across different environments. Key concepts include:

1. Dockerfile: A text file that contains instructions for building a Docker image.
2. Docker Image: A lightweight, standalone, and executable package that contains all the necessary dependencies to run an application.
3. Docker Container: An instance of a Docker image that runs as a process on the host machine.
4. Docker-Compose: It is a tool for defining and running multi-container applications. It is the key to unlocking a streamlined and efficient development and deployment experience.

**Benefits of using Docker include:**
 
 1. Consistency: Ensures that applications run the same way in development, testing, and production environments.
 2. Isolation: Each container runs independently, preventing conflicts between dependencies.
 3. Portability: Docker images can be easily shared and deployed across different platforms.

### Section 2: Docker Hub:

Docker Hub is a cloud-based repository for storing, sharing, and distributing Docker images. Key features include:

1. Public and Private Repositories: Docker Hub allows users to create both public and private repositories to manage Docker images.
2. Versioning: Docker Hub supports versioning for images, making it easy to track changes and roll back to previous versions.
3. Automated Builds: Docker Hub can automatically build Docker images from source code repositories such as GitHub, GitLab, and Bitbucket.

Creating a Docker Hub account and repository is straightforward, allowing developers to share their Docker images with team members and the wider community.

### Section 3: GitHub Actions for CI/CD:

GitHub Actions is a powerful CI/CD platform built into GitHub, enabling developers to automate workflows directly from their repositories. Key features include:

1. YAML-based Workflows: GitHub Actions workflows are defined using YAML syntax, making them easy to read and maintain.
2. Trigger Events: Workflows can be triggered by various events, such as push, pull request, or schedule.
3. Marketplace Integrations: GitHub Actions provides a marketplace of pre-built actions that can be used to automate common tasks.

### Section 4: Setting up CI/CD Pipeline with GitHub Actions:

Setting up a CI/CD pipeline with GitHub Actions involves defining workflows that automate tasks such as building, testing, and deploying applications. A typical CI/CD pipeline may include the following steps:

1. Build: Compile the application code and create a Docker image.
2. Test: Run unit tests, integration tests, and other automated tests to ensure code quality.
3. Deploy: Deploy the Docker image to a target environment, such as a staging or production server.

GitHub Actions integrates seamlessly with Docker Hub, allowing developers to push Docker images to repositories as part of their CI/CD workflows.

### Section 5: Best Practices and Tips:

To ensure the success of CI/CD pipelines using Docker, Docker-Compose, Docker Hub, and GitHub Actions, it's essential to follow best practices:

1. Optimize Dockerfile: Keep Dockerfiles concise and minimize the number of layers to improve build performance.
2. Security Considerations: Regularly scan Docker images for vulnerabilities and apply security patches promptly.
3. Pipeline Optimization: Continuously monitor and optimize CI/CD pipelines for speed, reliability, and efficiency.


### Section 6: Real-World Use Cases:

## Building the react app and running using nginx:

***Project Structure:***

```tree
frontend/
├── public/
│   ├── index.html
├── src/
│   ├── components/
│   │   ├── App.js
│   │   └── ...
│   ├── index.js
│   └── ...
├── Dockerfile
├── .dockerignore
├── package.json
├── yarn.lock
├── entrypoint.sh
└── ...
```

- **Dockerfile:** Contains instructions to build the Docker image for the frontend React application.
- **.dockerignore:** Specifies files and directories to be excluded from the Docker build context.

### Dockerfile
```yaml
# Builder Stage
FROM node:21.6.0 as builder

WORKDIR /app

# Copy only the package files first to leverage Docker layer caching
COPY package.json yarn.lock ./

# Update package lists and upgrade packages
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y curl

# Install dependencies without husky hooks
RUN yarn install --ignore-scripts

# Copy the entire application
COPY . .

# Build the application
RUN yarn build

# Production Stage
FROM nginx:stable-perl

WORKDIR /usr/share/nginx/html

# Create the necessary directory and set permissions
RUN mkdir -p /var/cache/nginx/client_temp && \
    chown -R nginx:nginx /var/cache/nginx

RUN mkdir -p /var/run/nginx && \
    chown -R nginx:nginx /var/run/nginx

# Remove default Nginx contents
RUN rm -rf ./*

# Copy the built application from the builder stage
COPY --from=builder /app/dist .

# Copy the entrypoint script
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# Set the entrypoint
ENTRYPOINT ["/entrypoint.sh"]

USER root

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

The Dockerfile provided compiles the production version of the source code by executing the command:
```console
RUN yarn build
```
Afterwards, it places the resulting build into the directory used by Nginx to serve static pages on base dir (e.g ***/usr/share/nginx/html***):

```console
COPY --from=builder /app/dist .
```

***Github Actions Workflow Frontend*** 

Add a workflow file in your frontend project e.g (*.github/workflow/build-and-push.yml*)

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - master
      - feature/*

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Determine Docker Tag
        id: tag
        run: |
          if [[ $GITHUB_REF == 'refs/heads/master' ]]; then
            echo 'DOCKER_TAG=latest' >> $GITHUB_ENV
          elif [[ $GITHUB_REF == 'refs/heads/feature/'* ]]; then
            echo "DOCKER_TAG=feature-${GITHUB_REF#refs/heads/feature/}" >> $GITHUB_ENV
          else
            echo "DOCKER_TAG=unknown" >> $GITHUB_ENV
          fi

      - name: Print Docker tag
        run: |
          echo "Docker Tag: ${{ env.DOCKER_TAG }}"

      - name: Build Docker image
        run: docker build -t docker-hub-repo/frontend:${{ env.DOCKER_TAG }} .

      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKERHUB_TOKEN }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin

      - name: Push Docker image to Docker Hub
        run: docker push docker-hub-repo/frontend:${{ env.DOCKER_TAG }}
```
The GitHub Actions workflow provided utilizes the Dockerfile located within the project directory to construct the Docker image.
```console
docker build -t docker-hub-repo/frontend:${{ env.DOCKER_TAG }} 
```

 After the image is built, the action proceeds to push it to the Docker Hub repository using the command:

```console
docker push docker-hub-repo/frontend:${{ env.DOCKER_TAG }}
```

This command instructs Docker to push the built image tagged with a specific version, identified by the environment variable DOCKER_TAG, to the specified Docker Hub repository.

## Backend Django Application:

***Project Structure:***

```tree
backend/
├── myapp/
│   ├── models.py
│   ├── views.py
│   └── ...
├── manage.py
├── Dockerfile
├── .dockerignore
├── requirements.txt
├── entrypoint.sh
└── ...
```

- **Dockerfile:** Contains instructions to build the Docker image for the backend Django application.
- **.dockerignore:** Specifies files and directories to be excluded from the Docker build context.

### Dockerfile
```yaml
# === Stage 1: Python Build Stage ===
FROM python:3.9.18-slim-bullseye AS python-build-stage

# Install Git LFS
RUN apt-get update \
    && apt-get install -y git-lfs \
    && git lfs install \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Set build environment to 'local' by default
ARG BUILD_ENVIRONMENT=local

# Set the working directory for the build stage
WORKDIR /build

# Copy only the requirements file to the build stage
COPY ./requirements.txt .

# Install required system dependencies for building Python packages
RUN apt-get update \
    && apt-get install --no-install-recommends -y \
        build-essential \
        libpq-dev \
        gettext \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create Python Dependency and Sub-Dependency Wheels.
RUN pip install --no-cache-dir --upgrade pip \
    && pip wheel --no-cache-dir --no-deps --wheel-dir /wheels -r requirements.txt


# === Stage 2: Python Run Stage ===
FROM python:3.9.18-slim-bullseye AS python-run-stage

# Set build environment to 'local' by default
ARG BUILD_ENVIRONMENT=local

# Set the working directory for the run stage
WORKDIR /app

# Copy the Python wheels from the build stage to the run stage
COPY --from=python-build-stage /wheels /wheels

# Install apt packages for the run stage
RUN apt-get update \
    && apt-get install --no-install-recommends -y \
        sudo \
        git \
        bash-completion \
        nano \
        ssh \
        curl \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create a non-root user and add it to sudoers
RUN groupadd --gid 1000 dev-user \
    && useradd --uid 1000 --gid dev-user --shell /bin/bash --create-home dev-user \
    && echo "dev-user ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/dev-user \
    && chmod 0440 /etc/sudoers.d/dev-user

# Install required system dependencies for the run stage
RUN apt-get update \
    && apt-get install --no-install-recommends -y \
        libpq-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && pip install --no-cache-dir --upgrade pip

COPY ./requirements.txt .

# Install Python dependencies from wheels
RUN pip install --no-cache-dir --find-links=/wheels -r requirements.txt \
    && rm -rf /root/.cache \
    && rm -rf /wheels

# Copy entrypoint and start scripts
COPY ./entrypoint.sh /entrypoint.sh

# Copy application code to the working directory
COPY . .

# Set the entrypoint
ENTRYPOINT ["/entrypoint.sh"]

```

This Dockerfile design called multi-stage builds. By separating the build process into distinct stages, you can take advantage of Docker's layer caching mechanism to improve build performance and reduce image size.

1. ***Builder Stage***: This stage is responsible for preparing the dependencies and artifacts needed for your backend application. By downloading all the wheel files and performing any other build-time operations here, you create a cache layer of the project dependencies.

2. ***Runner Stage***: In this stage, you copy the necessary artifacts, such as the wheel files, from the builder stage and then install them. Since the wheel files are already downloaded and available, Docker can reuse the cached layers from the builder stage, reducing consecutive build times.

This approach effectively minimizes the need to reinstall dependencies during each build, as long as the dependencies listed in requirements.txt remain unchanged. It's particularly beneficial for projects with large dependency trees or slow internet connections, as it helps optimize build performance.

By using multi-stage builds, you can streamline your Dockerfile and improve the efficiency of your Docker image builds. Additionally, it helps maintain consistency and repeatability across different environments.

 
***Github Actions Workflow Backend*** 

Add a workflow file in your backend project e.g (*.github/workflow/build-and-push.yml*)

```yaml
name: Build and Push Docker Image

on:
  push:
    branches:
      - master
      - feature/*

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Determine Docker Tag
        id: tag
        run: |
          if [[ $GITHUB_REF == 'refs/heads/master' ]]; then
            echo 'DOCKER_TAG=latest' >> $GITHUB_ENV
          elif [[ $GITHUB_REF == 'refs/heads/feature/'* ]]; then
            echo "DOCKER_TAG=feature-${GITHUB_REF#refs/heads/feature/}" >> $GITHUB_ENV
          else
            echo "DOCKER_TAG=unknown" >> $GITHUB_ENV
          fi

      - name: Print Docker tag
        run: |
          echo "Docker Tag: ${{ env.DOCKER_TAG }}"

      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKERHUB_TOKEN }}" | docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin


      - name: Build and Push Docker image
        uses: docker/build-push-action@v2
        with:
          context: .
          file: ./Dockerfile
          tags: docker-hub-repo/backend:${{ env.DOCKER_TAG }}
          push: true
        env:
          DOCKERHUB_USERNAME: ${{ secrets.DOCKERHUB_USERNAME }}
          DOCKERHUB_TOKEN: ${{ secrets.DOCKERHUB_TOKEN }}
```
The GitHub Actions workflow we have added is using the Dockerfile located in the project directory to build the Docker image. After building the image, it pushes the image to a Docker Hub repository. Let's break down the important parts:

1. ***Building the Docker Image***:
    - ***context***: .: Specifies the build context, which is the path to the directory containing the Dockerfile. In this case, it's set to the root of the project directory.
    - ***file***: ./Dockerfile: Specifies the path to the Dockerfile within the build context.
    - ***tags***: docker-hub-repo/backend:${{ env.DOCKER_TAG }}: Tags the built image with a specific tag. The tag is derived from the DOCKER_TAG environment variable.
    - ***push***: true: Indicates that the built image should be pushed to a remote registry after the build process completes.

2. ***Docker Hub Credentials***:
    - ***DOCKERHUB_USERNAME*** and ***DOCKERHUB_TOKEN*** are environment variables used to authenticate with Docker Hub for pushing the image. They are retrieved from GitHub Secrets to keep sensitive information secure.

This workflow automates the process of building and pushing Docker images to a Docker Hub repository whenever changes are pushed to the repository. It's a convenient way to manage Docker images as part of a continuous integration/continuous deployment (CI/CD) pipeline.

***Ensure that Docker Hub credentials are stored as secrets in the GitHub repository.***


### Docker Compose:

Create a docker-compose.yml file to define the services and configurations for running the containerized applications.

```yaml
version: '3.4'

services:
  frontend:
    image: docker-hub-repo/frontend:${FRONTEND_IMAGE_VERSION:-latest}
    container_name: frontend
    env_file:
      - .docker-env
    depends_on:
      - backend
    ports:
      - "80:80"

  backend:
    image: docker-hub-repo/backend:${BACKEND_IMAGE_VERSION:-latest}
    container_name: backend
    env_file:
      - .docker-env
    command:
      - /start
    ports:
      - "8000:8000"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health_check/"]
      interval: 40s
      timeout: 60s
      retries: 3
      start_period: 60s

  celery:
    image: docker-hub-repo/backend:${BACKEND_IMAGE_VERSION:-latest}
    container_name: celery
    command: celery -A app-name worker --loglevel=info
    env_file:
      - .docker-env

  celery-beat:
    image: docker-hub-repo/backend:${BACKEND_IMAGE_VERSION:-latest}
    container_name: celery-beat
    command: celery -A app-name beat --loglevel=info
    env_file:
      - .docker-env
    depends_on:
      - celery

  redis:
    image: "redis:alpine"
    container_name: redis
    ports:
      - "6379:6379"

networks:
  default:
    name: fg-rms-network
    external: true

```
The provided docker-compose.yml file sets up five services, each corresponding to a component of the application:

1. ***Frontend*** (container to serve frontend)
2. ***Backend*** (container to serve backend)
3. ***Celery*** (container to run long running task)
4. ***Celery-beat*** (container to run periodic tasks)
5. ***Redis*** (container as a message broker service for celery)

These services are configured to use Docker images retrieved directly from Docker Hub, abstracting away the source code details.


***Steps to deploy new changes***

1. Pull Latest Images: Pull the latest Docker images from Docker Hub to ensure that you have the most up-to-date versions of your containerized applications.
```console
docker-compose pull
```

2. Start Containers: Start the containers for your applications using Docker Compose. The --force-recreate flag ensures that all containers are recreated, and the --no-deps flag prevents Docker Compose from recreating linked containers.
```console
docker-compose up --force-recreate --no-deps -d
```
`


## Conclusion:

By following this project structure and incorporating GitHub Actions for CI/CD, developers can automate the build and deployment process of their frontend React and backend Django applications. Using Docker Compose, these containerized applications can be easily deployed and scaled on any environment, including virtual machines.
