
# User-Management-API

This project provides a simple user management API built with **TypeScript**, **Express**, and **MongoDB**. The API includes features to create, read, update, and delete users. Docker is used to manage both the development and production environments.



## Docker Setup

This project uses **Docker Compose** to orchestrate the services for both the development and production environments. The two services that are defined are:
- **MongoDB**: A NoSQL database to store user data.
- **API**: The Express.js server that interacts with MongoDB and handles API requests.

### Common Components
- **Volumes**: Used to persist MongoDB data.
- **Networks**: Isolated network to allow communication between the services (API and MongoDB).

---

## Development Environment

In the development setup, the goal is to enable rapid iteration and debugging. The Docker Compose file for development includes the following:

```yaml
services:
  mongo:
    image: mongo:latest
    container_name: mongodb
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
      MONGO_INITDB_DATABASE: user_management_db
    volumes:
      - mongo-data-dev:/data/db

  api:
    image: express-ts-api:v1.0.0-dev
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    container_name: user_management_api
    volumes:
      - ./backend:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    environment:
      PORT: 3000
      MONGODB_URI: mongodb://root:example@mongo:27017/user_management_db?authSource=admin
    depends_on:
      - mongo

volumes:
  mongo-data-dev:
```

### Key Points:
- **Hot Reloading**: The API service uses the `Dockerfile.dev` to enable hot reloading. We mount the backend code directly to the container using:
  ```yaml
  volumes:
    - ./backend:/app
    - /app/node_modules
  ```
  This allows developers to make changes in the source code, and the changes are instantly reflected inside the container.
  
- **MongoDB Data Persistence**: The `mongo-data-dev` volume stores MongoDB data during development, ensuring that changes persist even if the container is restarted.

- **Dockerfile.dev**: This Dockerfile is tailored for development. It includes `npm run dev` to start the API in development mode, allowing for continuous development and testing without needing to rebuild the container.

### Running dev compose file:

```bash
docker compose -f dev.compose.yaml up -d --build 
```

### Stopping dev compose file:

```bash
docker compose -f dev.compose.yaml down 
```
---

## Production Environment

The production environment is more optimized for performance and reliability. It uses a multi-stage build to keep the image size small and ensure that unnecessary files, like development dependencies, are not included.

```yaml
services:
  mongo:
    image: mongo:latest
    container_name: mongodb
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
      MONGO_INITDB_DATABASE: user_management_db
    volumes:
      - mongo-data:/data/db
    networks:
      - user_management_network

  api:
    image: panchanandevops/express-ts-api:v4.0.0
    build:
      context: ./backend
      dockerfile: Dockerfile.prod
    container_name: user_management_api
    ports:
      - "3000:3000"
    environment:
      PORT: 3000
      MONGODB_URI: mongodb://root:example@mongo:27017/user_management_db?authSource=admin
    depends_on:
      - mongo
    networks:
      - user_management_network

volumes:
  mongo-data:

networks:
  user_management_network:
    driver: bridge
```

### Key Points:
- **Production Image**: The `Dockerfile.prod` is used to build a production-ready image for the API. This ensures a clean and optimized environment for deployment.
  
- **Multi-Stage Build**: The production Dockerfile uses a multi-stage build to first build the application and then copy only the necessary files to the final image. This keeps the final image size smaller and avoids including development dependencies.

- **MongoDB Data Persistence**: In production, a persistent volume `mongo-data` ensures that the MongoDB data remains available even after the container is stopped or recreated.

- **Custom Network**: A custom network (`user_management_network`) ensures that both the `mongo` and `api` services can communicate with each other internally, but remain isolated from external services.

### Running prod compose file:

```bash
docker compose -f prod.compose.yaml up -d --build 
```

### Stopping prod compose file:

```bash
docker compose -f prod.compose.yaml down 
```

---

## Curl Commands for API Testing

You can interact with the API using the following `curl` commands to create, retrieve, update, and delete users.

### 1. POST Requests

#### Create a new user (Example 1)
```sh
curl -X POST http://localhost:3000/users \
-H "Content-Type: application/json" \
-d '{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123"
}'
```

#### Create a new user (Example 2)
```sh
curl -X POST http://localhost:3000/users \
-H "Content-Type: application/json" \
-d '{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "password456"
}'
```

### 2. GET Requests

#### Get all users
```sh
curl -X GET http://localhost:3000/users
```

#### Get a user by ID
Replace `USER_ID` with the actual ID of the user.
```sh
curl -X GET http://localhost:3000/users/USER_ID
```

### 3. PUT Requests

#### Update a user by ID
Replace `USER_ID` with the actual ID of the user.
```sh
curl -X PUT http://localhost:3000/users/USER_ID \
-H "Content-Type: application/json" \
-d '{
  "name": "Updated Name",
  "email": "updated@example.com",
  "password": "newpassword123"
}'
```

### 4. DELETE Requests

#### Delete a user by ID
Replace `USER_ID` with the actual ID of the user.
```sh
curl -X DELETE http://localhost:3000/users/USER_ID
```

These commands assume that your server is running on `localhost` on port `3000`. If your server is running on a different port or host, make sure to adjust the URL accordingly.

