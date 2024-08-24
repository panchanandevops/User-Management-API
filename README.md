# User-Management-API

docker pull k8s.gcr.io/ingress-nginx/controller:v1.10.1
minikube image load k8s.gcr.io/ingress-nginx/controller:v1.10.1

here are the `curl` commands for the POST, GET, PUT, and DELETE operations based on your Express API:

### 1. POST Requests

#### Create a new user (1)
```sh
curl -X POST http://panchanandevops.com:30452/users \
-H "Content-Type: application/json" \
-d '{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123"
}'
```

#### Create a new user (2)
```sh
curl -X POST http://panchanandevops.com:30452/users \
-H "Content-Type: application/json" \
-d '{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "password456"
}'
```

#### Create a new user (3)
```sh
curl -X POST http://panchanandevops.com:30452/users \
-H "Content-Type: application/json" \
-d '{
  "name": "Alice Smith",
  "email": "alice@example.com",
  "password": "password789"
}'
```

### 2. GET Requests

#### Get all users
```sh
curl -X GET http://panchanandevops.com:30452/users
```

#### Get a user by ID
Replace `USER_ID` with the actual ID of the user.
```sh
curl -X GET http://panchanandevops.com:30452/users/66c63abdda7164136f89c956
```

### 3. PUT Requests

#### Update a user by ID
Replace `USER_ID` with the actual ID of the user.
```sh
curl -X PUT http://panchanandevops.com:30452/users/66c63abdda7164136f89c956 \
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
curl -X DELETE http://panchanandevops.com:30452/users/66c63abdda7164136f89c956
```

These commands assume that your server is running on `panchanandevops.com` on port `80`. If your server is running on a different port or host, make sure to adjust the URL accordingly.


