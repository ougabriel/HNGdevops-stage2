# HNGdevops-stage2

###TASK 
DevOps Stage 2: Dockerized Full Stack Web Application Deployment
In this stage, you’ll deploy a provided full stack web application (React frontend and FastAPI + PostgreSQL backend) using Docker containers and proxy both services to run on the same port using Traefik or Nginx Proxy Manager.
Requirements:
1. Fork the Repository:
   Create a fork of this repository to add the necessary Docker and proxy configuration files.
2. Dockerization:
   Write Dockerfiles to containerize both the React frontend and FastAPI backend. Ensure each service can be built and run locally in their respective containers.
3. Configure Traefik, HaProxy or Nginx Proxy Manager to:
   - Serve the frontend and backend on the same host machine port (80).
   - Serve the frontend on the root (/).
   - Proxy /api on the backend to /api on the main domain.
   - Proxy /docs on the backend to /docs on the main domain.
   - Proxy /api on the backend to /api on the main domain.
4. Database Configuration:
   Configure the application to use a PostgreSQL database. Ensure the database is properly set up and connected.
5. Adminer Setup:
   Configure Adminer to run on port 8080. Ensure Adminer is accessible via the subdomain db.domain and is properly connected to the postgres database.
6. Proxy Manager Setup:
   Configure the proxy manager to run on port 8090. Ensure the proxy manager is accessible via the subdomain proxy.domain.
7. Cloud Deployment:
   Deploy your Dockerized application to an AWS EC2 instance. Set up a domain for your application. If you don't have a domain, get a free subdomain from Afraid DNS. Configure HTTP to redirect to HTTPS. Configure www to redirect to non-www.
Additional Instructions:
There is a minimal README with simple instructions on starting the application in the repository. However, you are required to improve all the README files with well-detailed instructions on how to deploy the app, both manually and using Docker.
Testing and Evaluation:
Your hosted application will be tested to ensure it meets all the requirements. Your repository will be tested to ensure it works correctly after running docker-compose up -d.

Submission Mode:
Submit your task through the designated submission form. Ensure you've:
- Double-checked all requirements and acceptance criteria.
- Provided a link to your deployed application and forked repository in the submission form.
- Thoroughly reviewed your work to ensure accuracy, functionality, and adherence to the specified guidelines before submission.



#1. Fork the Repository:
   
   - Create a new Azure VM
   - Update the VM `sudo apt update`
   - clone the repo `git clone https://github.com/hngprojects/devops-stage-2`
   - `cd` into the repo `cd devops-stage2`

#2. Dockerize the App
    Create two Dockerfiles
      - Create the `Dockerfile` for the `frontend`
          a. `cd` into the frontend and create the file
            ```bash
            # Use the latest official Node.js image as a base
            FROM node:latest
            # Set the working directory
            WORKDIR /app
            # Copy the application files
            COPY . .
            # Install dependencies
            RUN npm install
            # Expose the port the development server runs on
            EXPOSE 5173
            # Run the development server
            CMD ["npm", "run", "dev", "--", "--host"]
            ```
       - Create the  `Dockerfile` for the `backend`
     b. `cd` into the backend and create the file
         
               ```bash
                 # Use the latest official Python image as a base
                FROM python:latest
                
                # Install Node.js and npm
                RUN apt-get update && apt-get install -y \
                    nodejs \
                    npm
                
                # Install Poetry using pip
                RUN pip install poetry
                
                # Set the working directory
                WORKDIR /app
                
                # Copy the application files
                COPY . .
                
                # Install dependencies using Poetry
                RUN poetry install
                # Expose the port FastAPI runs on
                EXPOSE 8000
                # Run the prestart script and start the server
             ```
   c. Push the `Dockerfiles` into a Dockerhub account
   
#3 Setup the database
     - `cd` to the `backend`
     in the backend we will configure the database
     - Install the packages 
         ```bash 
        sudo apt update
        sudo apt install postgresql postgresql-contrib
        ```
      - create a user and database
      
      ```bash
      #1. login
      sudo -i -u postgres
      psql
      
      #2. create the user and database
      
      CREATE USER app WITH PASSWORD 'my_password';
     CREATE DATABASE app;
      \c app
      GRANT ALL PRIVILEGES ON DATABASE app TO app;
      GRANT ALL PRIVILEGES ON SCHEMA public TO app;
      
      #3. Exit
      \q
      exit
      ```
  - Update `.env` variable in the `backend`
    `vi` into the file and edit the following content inside it to match the database
      ```bash
        POSTGRES_SERVER=localhost
        POSTGRES_PORT=5432
        POSTGRES_DB=app
        POSTGRES_USER=app
        POSTGRES_PASSWORD=my_password
      ```
#4: Configure `nginx proxy manager` , and `adminer` using `docker-compose`

  - Install `docker` and `docker-compose`
    
    ```bash

    #1. Update
    sudo apt-get update

    #2. Install packages
    sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

    #3. Add docker GPG key and repo
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
      echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

    #4. update and install
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io

    #5. install docker-compose
    sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep -Po '"tag_name": "\K.*?(?=")')/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

    #6. permissions
    sudo chmod +x /usr/local/bin/docker-compose

    #7. Create and add docker group
    sudo groupadd docker
    sudo usermod -aG docker $USER 

    #8. Start, enable and Check versions
    sudo systemctl start docker
    sudo systemctl enable docker
    docker --version
    docker-compose --version
    ```
    
 - Create the `docker-compose` file and set it up
    
           ```bash
              version: '3.8'
              services:
                backend:
                  build:
                    context: ./backend
                  container_name: fastapi_app
                  ports:
                    - "8000:8000"
                  depends_on:
                    - db
                  env_file:
                    - ./backend/.env
              
                frontend:
                  build:
                    context: ./frontend
                  container_name: nodejs_app
                  ports:
                    - "5173:5173"
                  env_file:
                    - ./frontend/.env
              
                db:
                  image: postgres:latest
                  container_name: postgres_db
                  ports:
                    - "5432:5432"
                  volumes:
                    - postgres_data:/var/lib/postgresql/data
                  env_file:
                    - ./backend/.env
              
                adminer:
                  image: adminer
                  container_name: adminer
                  ports:
                    - "8080:8080"
              
                proxy:
                  image: jc21/nginx-proxy-manager:latest
                  container_name: nginx_proxy_manager
                  ports:
                    - "80:80"
                    - "443:443"
                    - "81:81"
                  environment:
                    DB_SQLITE_FILE: "/data/database.sqlite"
                  volumes:
                    - ./data:/data
                    - ./letsencrypt:/etc/letsencrypt
                  depends_on:
                    - db
                    - backend
                    - frontend
                    - adminer
              
              volumes:
                postgres_data:
                data:
                letsencrypt:
              networks:
                webnet:
                dbnet:
            ```     

        #
