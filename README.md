## Multi-Container Docker Application with CI/CD: Calculator App Project

#### Complete Project Instructions: [DevOps Foundations Course/Project](https://github.com/shiftkey-labs/DevOps-Foundations-Course/tree/master/Project)

#### Submission by - **Minh Nguyen**

### Project Overview

- **Brief project description:** What is the purpose of your application?
The application is the fullstack calculator web app with frontend and backend. The app is containerized with Docker and continuously integrated with Github Action


- **Which files are you implmenting? and why?:**
    - frontend/Dockerfile: Docker file for building image of the frontend React application
    - frontned/default.conf: Nginx server configuration file for serving frontend content 
    - backend/Dokcerfile: Docker file for building image of the backend Python application

### Docker Implementation

**Explain your Dockerfiles:**

- **Backend Dockerfile** (Python API):
    - The "FROM python:3.9-slim" utilizes the base image with python run time installed to run the application
    - The "WORKDIR /app" set the current working directory to app directory
    - The "COPY . ." copies all the files that are required to install dependencies to working dir, and run the application
    - The "RUN pip install -r requirements.txt" installed the dependencies for the app to run (Flask, cors, pytest)
    - The "EXPOSE 5000" make the port 5000 available
    - The "CMD [ "python", "app.py" ]" run the python backend application

- **Frontend Dockerfile** (React App):
    - The "FROM node:14-alpine AS build" utilizes the base image with Node JS run time installed to run the application, this set the current stage as build
    - The "WORKDIR /app" set the current working directory to app directory
    - The "COPY package.json ." copies the package.json in the working dir (/app) to install dependencies
    - The "RUN npm install" install app dependencies
    - The "COPY . ." copies all the files that are required to run the application to working dir
    - The "RUN npm run build" build the frontend application. The build artifact will be stored in default /build dir inside current working dir
    - The "FROM nginx:alpine" utilizes the base image of this stage with nginx server to serve the frontend app, multple FROM indicate the Dockerfile is multi-stage
    - The "COPY --from=build /app/build /usr/share/nginx/html" copies the build artifact from previous stage "build" to nginx html folder to serve content
    - The "COPY default.conf /etc/nginx/conf.d/default.conf" copies the nginx default config file to the nginx config folder
    - The "EXPOSE 3000" make the port 3000 available
    - The "CMD ["nginx", "-g", "daemon off;"]" start Nginx

**Use this section to document your choices and steps for building the Docker images.**
- Frontend build image instruction: docker build -t frontend . 
- Frontend run container instruction: docker run -p 3000:3000 --name frontend frontend
- Backend build image instruction: docker build -t backend . 
- Backend run container instruction: docker run -p 5000:5000 --name backend backend

### Docker Compose YAML Configuration

**Break down your `docker-compose.yml` file:**

- **Services:** 
    - frontend: service for frontend React application
    - backend: service for backend Python application
- **Networking:** 
    - In the context of this calculator app, there is no need to create a bridge network to facilites communication between the frontend app and backend app. 
    - When the user access the url of the frontend, the nginx server serve the content to the browser and the communication to the backend is from browser NOT the frontend app, so there is no interaction between frontend and backend
- **Volumes:** 
    - No Volume needed
- **Environment Variables:**
    - No

**Use this section to explain how your services interact and are configured within `docker-compose.yml`.**
    -  build:
        context: ./frontend
        dockerfile: Dockerfile
    Set frontend, backend context (Dockerfile location)

    -  ports: 
        - "3000:3000"
    Map the host port to the container port


### CI/CD Pipeline (YAML Configuration)

**Explain your CI/CD pipeline:**

- What triggers the pipeline 
    - push to the main branch
    - pull request to the main branch
    - manual triggers
- What are the different stages (build, test, deploy)?
    - build, test, and publish to docker hub
- How are Docker images built and pushed to a registry (if applicable)?
    - Uses built in Action: Build and push Docker images

**Use this section to document your automated build and deployment process.**
- frontend-test stage:
    - "uses: actions/checkout@v4" checkout code
    - "uses: actions/setup-node@v4.1.0" set up Node JS
    - "run: npm install" install dependencies
    - Run tests frontend

- frontend-build stage:
    - "needs: frontend-test" wait for test pass before building
    - "uses: actions/checkout@v4" checkout code
    - "uses: actions/setup-node@v4.1.0" set up Node JS
    - "npm install" install dependencies
    - "run: npm run build" build the app
      
- backend stage:
    - "uses: actions/checkout@v4" checkout code
    - "uses: actions/setup-python@v5.3.0" set up Python
    - "run: pip install -r requirements.txt" install dependencies
    - "run: python test_app.py" test the app
      
- docker-fe stage:
    - "needs: frontend-build" wait for build before publish to docker hub
    - "uses: docker/setup-qemu-action@v3" Set up QEMU
    - "uses: adocker/setup-buildx-action@v3" Set up Docker Buildx
    - "uses: docker/login-action@v3" Login to Docker Hub
    - "uses: docker/build-push-action@v6" Build and push frontend image to docker hub
      
- docker-be stage:
    - "needs: backend" wait for test before publish to docker hub
    - "uses: docker/setup-qemu-action@v3" Set up QEMU
    - "uses: docker/setup-buildx-action@v3" Set up Docker Buildx
    - "uses: docker/login-action@v3" Login to Docker Hub
    - "uses: docker/build-push-action@v6" Build and push frontend image to docker hub

### Assumptions

- List any assumptions you made while creating the Dockerfiles, `docker-compose.yml`, or CI/CD pipeline. 
    - The testing of frontend have some error so npm test is expected to fail
    - There is no need for networking between frontend and backend

### Lessons Learned

- What challenges did you encounter while working with Docker and CI/CD?
    - In the context of this calculator app, there is no need to create a bridge network to facilites communication between the frontend app and backend app. 
    - When the user accesses the URL of the frontend, the nginx server serves the content to the browser and the communication to the backend is from the browser NOT the frontend app, so there is no interaction betweenthe  frontend and backend
    - Learn to use built-in marketplace github actions
- What did you learn about containerization and automation?
    - Improve significantly the development and testing

**Use this section to reflect on your experience and learnings when implementing this project.**
    - Learn more about the Docker networking
    - Learn about multistage Dockerfile
    - Learn about networking between frontend app and backend app

### Future Improvements

- How could you improve your Dockerfiles, `docker-compose.yml`, or CI/CD pipeline? 
    - Use volumes and complex networking with containers 
    - Implement CD pipeline to deploy app host on Cloud

**Use this section to brainstorm ways to enhance your project.**
    - Try all volume types in Docker 
    - Implement a more complex network in Docker with many containers
    - CD to deploy on EC2 
