# User Management API

This project is a basic **User Management API** built with **TypeScript**, **Express.js**, and **MongoDB**. The project supports full CRUD operations on a `User` model and connects to a MongoDB database using `Mongoose`. The API is designed to be containerized using Docker for both **development** and **production** environments.

testing2444

## Project Structure

```
backend
├── Dockerfile.dev
├── Dockerfile.prod
├── package.json
├── package-lock.json
├── README.md
├── src
│   ├── index.ts
│   ├── models
│   │   └── user.ts
│   └── routes
│       └── user.ts
└── tsconfig.json
```

## API Endpoints

- **POST /users**: Create a new user
- **GET /users**: Fetch all users
- **GET /users/:id**: Get a specific user by ID
- **PUT /users/:id**: Update a user by ID
- **DELETE /users/:id**: Delete a user by ID

## Environment Variables

- `MONGODB_URI`: MongoDB connection string
- `PORT`: Port number on which the server will run

## Setting Up Docker for Development and Production

We have two separate Dockerfiles for **development** (`Dockerfile.dev`) and **production** (`Dockerfile.prod`) environments. This separation allows for optimized workflows in both environments.

### Development Dockerfile (`Dockerfile.dev`)

This Dockerfile is designed for development purposes. It provides a development-friendly environment where changes to the source code are reflected immediately in the running container through **hot-reloading**.

```dockerfile
# Use a Node.js base image
FROM node:22-bullseye-slim

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json to the container
COPY package*.json ./

# Install dependencies
RUN npm install

# Expose the port your application runs on
EXPOSE 3000

# Command to start the application with hot reload
CMD ["npm", "run", "dev"]
```

#### Explanation of each step:

1. **`FROM node:22-bullseye-slim`**:
   - We use the official **Node.js 22** image with a lightweight Debian-based variant (`bullseye-slim`) as the base image. This provides us with a minimal environment, reducing the container's size.
2. **`WORKDIR /app`**:

   - This sets the working directory for any further `RUN`, `COPY`, or `CMD` instructions. All following instructions will operate inside `/app` in the container.

3. **`COPY package*.json ./`**:

   - We copy the `package.json` and `package-lock.json` files into the container. This ensures that the list of dependencies is available for installation.

4. **`RUN npm install`**:

   - Installs all the project dependencies. Running this step after copying the `package.json` ensures the dependencies are installed first, avoiding redundant re-installations when the application code changes.

5. **`EXPOSE 3000`**:

   - Exposes port **3000**, which is the port where our Express.js API will be accessible inside the container.

6. **`CMD ["npm", "run", "dev"]`**:
   - The container starts by running `npm run dev`, which triggers the **development** mode. This mode typically uses **hot-reloading**, which monitors code changes and restarts the server automatically for a smooth developer experience.

### Production Dockerfile (`Dockerfile.prod`)

This Dockerfile is built for running the application in a **production** environment. It includes a multi-stage build process to ensure a lightweight and optimized container.

```dockerfile
# Stage 1: Build the application
FROM node:22-bullseye-slim AS build

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json to the container
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application code to the container
COPY . .

# Build the application
RUN npm run build

# Stage 2: Run the application
FROM node:22-bullseye-slim

# Set the working directory in the container
WORKDIR /app

# Copy only the necessary files from the build stage
COPY --from=build /app/dist /app

# Copy node_modules from build stage
COPY --from=build /app/node_modules /app/node_modules

# Expose the port your application runs on
EXPOSE 3000

# Command to run your application
CMD ["node", "index.js"]
```

#### Explanation of each step:

1. **Multi-stage build**:
   - The Dockerfile is divided into two stages: **build** and **run**.
2. **Stage 1: Build the application**:

   - **`FROM node:22-bullseye-slim AS build`**: We use the Node.js image again, and this stage will be used for building the application.
   - **`WORKDIR /app`**: Sets the working directory to `/app`.
   - **`COPY package*.json ./`**: Copies the `package.json` and `package-lock.json` files for dependency installation.
   - **`RUN npm install`**: Installs the project dependencies.
   - **`COPY . .`**: Copies the rest of the application code into the container.
   - **`RUN npm run build`**: Builds the application, typically creating an optimized version of the app in a `/dist` directory.

3. **Stage 2: Run the application**:
   - **`FROM node:22-bullseye-slim`**: A fresh Node.js image is used for the production stage to keep the container lightweight.
   - **`WORKDIR /app`**: Sets the working directory to `/app`.
   - **`COPY --from=build /app/dist /app`**: Copies the built application from the `build` stage. Only the necessary files are copied.
   - **`COPY --from=build /app/node_modules /app/node_modules`**: Also copies the `node_modules` folder from the `build` stage, ensuring all dependencies are available.
   - **`EXPOSE 3000`**: Exposes port **3000** to allow external access to the application.
   - **`CMD ["node", "index.js"]`**: Starts the production server using the built version of the app with the `node` command.

## Usage

### Development Environment

To build and run the development environment, you can use the following command with **Docker Compose** or run the development Dockerfile directly.

1. **Build the Docker image for development**:

   ```bash
   docker build -f Dockerfile.dev -t user-management-dev .
   ```

2. **Run the container**:
   ```bash
   docker run -p 3000:3000 user-management-dev
   ```

The development environment will use **hot-reloading** to reflect any changes you make in the codebase.

### Production Environment

For production, a two-stage build process is used to optimize the final Docker image.

1. **Build the production Docker image**:

   ```bash
   docker build -f Dockerfile.prod -t user-management-prod .
   ```

2. **Run the production container**:
   ```bash
   docker run -p 3000:3000 user-management-prod
   ```

This image contains only the necessary files to run the application in production, making it more lightweight and efficient.
