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
