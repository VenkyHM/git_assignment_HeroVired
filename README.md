# git_assignment_HeroVired

******************************************
SKILL TEST-1 IMPORTANT POINTS
*****************************************
Create an EC2 instance and update as mentioned below.

sudo -i
sudo yum update -y
sudo yum unstall docker -y
sudo systemctl start docker
sudo systemctl enable docker

-----------------------------------------------------
Creare three folders as mentioned below
mkdir -p submission/{user-service,product-servicec,gateway-service}
Three separate files in each foldes 1. Dockerfile 2.package.json & 3.index.js
there should be a docker-compose.yml in submission folder.
----------------------------------------------------------------------------
Contents should be present as below.

docker-compose.yml
version: "3.8"

services:
  user-service:
    build: ./user-service
    ports:
      - "3000:3000"
    networks:
      - app-network

  product-service:
    build: ./product-service
    ports:
      - "3001:3001"
    networks:
      - app-network

  gateway-service:
    build: ./gateway-service
    ports:
      - "3003:3003"
    networks:
      - app-network
    depends_on:
      - user-service
      - product-service

networks:
  app-network:
    driver: bridge
-----------------------------------------------------------------------------------
Dockerfile's 

--------------------------
# Use Node.js base image
FROM node:18

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy rest of the code
COPY . .

# Expose service port (change per service)
EXPOSE 3000

# Start the app
CMD ["node", "index.js"]
----------------------------------

FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

EXPOSE 3001
CMD ["node", "index.js"]

----------------------------------
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3003
CMD ["node", "index.js"]


----------------------------------
packagefile
---------------------------------
{
  "name": "user-service",
  "version": "1.0.0",
  "description": "User microservice",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}

---------------------------------------
{
  "name": "product-service",
  "version": "1.0.0",
  "description": "Product microservice",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
----------------------------------------------
{
  "name": "gateway-service",
  "version": "1.0.0",
  "description": "Gateway microservice",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "axios": "^1.6.0"
  }
}

----------------------------------------
index.js file
-----------------------------------------
user-service
--------------------------
const express = require("express");

const app = express();
const PORT = 3000;

app.get("/", (req, res) => {
  res.json({ message: "User Service is running" });
});

app.get("/users", (req, res) => {
  res.json([
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" }
  ]);
});

app.listen(PORT,'0.0.0.0' , () => {
  console.log(`User Service running on port ${PORT}`);
});
-------------------------------------------------------------
product-service
------------------------------------------------------------
const express = require("express");

const app = express();
const PORT = 3001;

app.get("/", (req, res) => {
  res.json({ message: "Product Service is running" });
});

app.get("/products", (req, res) => {
  res.json([
    { id: 1, name: "Laptop" },
    { id: 2, name: "Phone" }
  ]);
});

app.listen(PORT, () => {
  console.log(`Product Service running on port ${PORT}`);
});

---------------------------------------------------------
gateway-service
--------------------------------------------------------
const express = require("express");
const axios = require("axios");

const app = express();
const PORT = 3003;

app.get("/", (req, res) => {
  res.json({ message: "Gateway Service is running" });
});

// Fetch users from user-service
app.get("/users", async (req, res) => {
  try {
    const response = await axios.get("http://user-service:3000/users");
    res.json(response.data);
  } catch (error) {
    res.status(500).json({ error: "Cannot fetch users" });
  }
});

// Fetch products from product-service
app.get("/products", async (req, res) => {
  try {
    const response = await axios.get("http://product-service:3001/products");
    res.json(response.data);
  } catch (error) {
    res.status(500).json({ error: "Cannot fetch products" });
  }
});

app.listen(PORT, () => {
  console.log(`Gateway Service running on port ${PORT}`);
});
-----------------------------------------------------------------------------
Once done run the below commands on submission folder

sudo curl -SL https://github.com/docker/compose/releases/download/v2.27.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose version

docker-compose up --build


**********************************************************
gateway improved code 
*********************************************************

const express = require('express');
const axios = require('axios');

const app = express();
app.use(express.json());

const port = 3003;

// Root route
app.get('/', (req, res) => {
  res.json({ message: 'Gateway Service is running' });
});

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'Gateway Service is healthy' });
});

// Get users
app.get('/api/users', async (req, res) => {
  try {
    const response = await axios.get('http://user-service:3000/users');
    res.json(response.data);
  } catch (error) {
    console.error('Users Error:', error.message);
    res.status(500).json({ error: 'Error fetching users', details: error.message });
  }
});

// Get products
app.get('/api/products', async (req, res) => {
  try {
    const response = await axios.get('http://product-service:3001/products');
    res.json(response.data);
  } catch (error) {
    console.error('Products Error:', error.message);
    res.status(500).json({ error: 'Error fetching products', details: error.message });
  }
});

// OPTIONAL: Only include if order-service exists

// Get orders
app.get('/api/orders', async (req, res) => {
  try {
    const response = await axios.get('http://order-service:3002/orders');
    res.json(response.data);
  } catch (error) {
    console.error('Orders Error:', error.message);
    res.status(500).json({ error: 'Error fetching orders', details: error.message });
  }
});

// Create order
app.post('/api/orders', async (req, res) => {
  try {
    const response = await axios.post(
      'http://order-service:3002/orders',
      req.body
    );
    res.json(response.data);
  } catch (error) {
    console.error('Create Order Error:', error.message);
    res.status(500).json({ error: 'Error creating order', details: error.message });
  }
});

app.listen(port, () => {
  console.log(`Gateway service running on port ${port}`);
});






