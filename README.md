# Hello World Node.js Application

This is a simple "Hello World" Node.js application. It serves as a basic example to demonstrate the setup and containerization of a Node.js application using Docker.

## Application Overview

The application consists of a single file, `app.js`, which creates a minimal HTTP server. Upon accessing the server, it responds with a "Hello, World!" message.

## Setup Instructions

### Prerequisites

Before running the application, ensure you have the following installed on your system:

- Node.js (https://nodejs.org/)
- Docker (https://www.docker.com/)

### Running Locally

1. Clone this repository to your local machine:

   ```bash
   git clone https://github.com/your-username/hello-world-node.git
   
2. Navigate to the project directory:
   cd hello-world-node
   
3. Install dependencies:
   npm install

4. Run the application:
   node app.js
   
Open your web browser and visit http://localhost:3000 to see the "Hello, World!" message.

### Docker Containerization
considering docker is installed in your system.
1. Build the Docker image:
docker build -t hello-world-node .

2. Run the Docker container:
docker run -p 3000:3000 hello-world-node

Visit http://localhost:3000 to see the "Hello, World!" message served from the Docker container.
