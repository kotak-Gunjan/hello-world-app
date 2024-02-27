 
Important Links:
GitHub: https://github.com/kotak-Gunjan/hello-world-app
Note: I have deployed the application on a Windows machine, so the documentation includes commands tailored for Windows support.

## Containerize an Application Using Docker:
Assuming, Docker and Node.js are already installed on your local system, let's containerize a Node.js "Hello World" application:
Step 1: Create a Simple Hello World Application
Create a file named app.js with the following content:
// app.js
    ```bash
    const http = require('http');

    const server = http.createServer((req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello, World!\n');
    });

    const PORT = 3000;
    server.listen(PORT, () => {
    console.log(`Server running at http://localhost:${PORT}/`);
    });

This Node.js application uses the built-in http module to create a simple HTTP server that responds with a "Hello, World!" message when accessed.

Step 2: Create a package.json for Node.js Application
Create a package.json file in the same directory as app.js:
    ```bash
    {
    "name": "docker-hello-world",
    "version": "1.0.0",
    "main": "app.js",
    "dependencies": {}
    }


Step 3: Containerize the Application
Create a file named Dockerfile in the same directory as app.js:
    ```bash
    # Use the official Node.js image as a base image
    FROM node:21

    # Set the working directory inside the container
    WORKDIR /usr/src/app

    # Copy package.json and package-lock.json to the working directory
    COPY package*.json ./

    # Install app dependencies
    RUN npm install

    # Copy the rest of the application code to the working directory
    COPY . .

    # Expose the port the app runs on
    EXPOSE 3000

    # Command to run the application
    CMD [ "node", "app.js" ]


This Dockerfile uses the official Node.js 21 Alpine image, sets the working directory to /app, copies the package.json and app.js files into the container, exposes port 3000, and sets the default command to run the application.

Step 4: Build and Run the Docker Image Locally
Open a terminal and navigate to the directory containing app.js, package.json, and Dockerfile. Then, run the following commands:
    ```bash
    # Build the Docker image
    docker build -t hello-world-app .

    # Run the Docker container
    docker run -p 3000:3000 hello-world-app


Now, a simple Node.js "Hello World" application is Dockerized. The application is accessible at http://localhost:3000/ on your local machine.
 
## Set Up Backstage:

I've successfully followed the steps outlined in the documentation at https://backstage.io/docs/getting-started/ to deploy Backstage on my Windows machine.
Prerequisites:
•	nvm 
•	Node modules.
•	Docker
•	Git
•	yarn

Step 1: 
To install the Backstage Standalone app, we make use of npx, a tool to run Node executables straight from the registry. This tool is part of your Node.js installation. Running the command below will install Backstage.
    ```bash
    npx @backstage/create-app@latest

The wizard will ask you for the name of the app, which will also be the name of the directory.

Step 2: Run the Backstage app.
    ```bash
    cd my-backstage-app
    yarn dev

as soon as the message [0] webpack compiled successfully appears, you can open a browser and directly navigate to your freshly installed Backstage portal at http://localhost:3000 .

Step3: Setting up authentication.
I have utilized GitHub and adhered to the instructions provided at https://backstage.io/docs/getting-started/configuration.
Go to https://github.com/settings/applications/new to create an OAuth App. The Homepage URL should point to Backstage's frontend, here it would be http://localhost:3000. The Authorization callback URL will point to the auth backend, which will be http://localhost:7007/api/auth/github/handler/frame refer to the below Image.
 
Figure 1: Github page to register new OAuth for Backstage
Take note of the Client ID and the Client Secret. Open app-config.yaml, and add your clientId and clientSecret to this file. It should end up looking like this:
    ```bash
    auth:
    # see https://backstage.io/docs/auth/ to learn about auth providers
    environment: development
    providers:
        github:
            development:
            clientId: YOUR CLIENT ID
            clientSecret: YOUR CLIENT SECRET

Now, Add sign-in option to the frontend, open packages/app/src/App.tsx and below the last import line, add:
    ```bash
    import { githubAuthApiRef } from '@backstage/core-plugin-api';
    import { SignInPage } from '@backstage/core-components';

Search for const app = createApp({ in this file, and below apis, add:
    ```bash
    components: {
    SignInPage: props => (
        <SignInPage
        {...props}
        auto
        provider={{
            id: 'github-auth-provider',
            title: 'GitHub',
            message: 'Sign in using GitHub',
            apiRef: githubAuthApiRef,
        }}
        />
    ),
    },

Restart Backstage from the terminal, there will be github login prompt in the welcome page.

Step 4: Setting up a GitHub Integration.
Generate a GitHub Personal Access Token with the "repo" and "workflow" scopes enabled during the token creation process. 
In the backstage root directory, update the  ‘app-config.local.yaml’ file by adding the following:
    ```bash
    integrations:
    github:
        - host: github.com
        token: ghp_urtokendeinfewinfiwebfweb # this should be the token from GitHub

now restart the backstage to apply these changes.
 
## Integrate Docker with Backstage:
To integrate the Backstage component into my containerized application, I registered the existing component by linking it to my GitHub repository. 

Click the "Analyze" button, and it will generate a pull request in your GitHub repository. This pull request includes the addition of the 'catalog-info.yaml' file. After the request is merged, your application component will be successfully registered on Backstage.

 
Figure 4: My hello world application integrated in backstage from github
 
Create a “CI/CD Pipeline”
Step1:
Include the 'docker-publish.yml' file in the '.github/workflows/' directory to activate a pipeline. This pipeline is designed to build and deploy the Docker image to the local Docker environment. Ensure that the action for this pipeline is set to manual.
    ```bash
    name: Docker

    # This workflow uses actions that are not certified by GitHub.
    # They are provided by a third-party and are governed by
    # separate terms of service, privacy policy, and support
    # documentation.

    on:
    workflow_dispatch:
        inputs:
        name:
            description: "When to run"
            default: "Now"

    jobs:
    build-and-deploy:
        runs-on: ubuntu-latest
        permissions:
        contents: read
        packages: write
        # This is used to complete the identity challenge
        # with sigstore/fulcio when running outside of PRs.
        id-token: write

        steps:
        - name: Checkout Repository
        uses: actions/checkout@v2

        - name: Build Docker Image
        run: |
            docker build -t gunjank05/hello-world-app:1.0.0 .
            echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
            docker push gunjank05/hello-world-app:1.0.0

        - name: Deploy to Local Docker
        run: |
            # Your deployment script to deploy to local Docker environment
            docker stop hello-world-app-container || true
            docker rm hello-world-app-container || true
            docker run -d --name hello-world-app-container -p 3000:3000 gunjank05/hello-world-app:1.0.0


Step 2:
To enable github actions from backstage application I have referred the steps from https://www.npmjs.com/package/@backstage/plugin-github-actions .
From your backstage root directory run the command :
# From your Backstage root directory
    ```bash
    yarn add --cwd packages/app @backstage/plugin-github-actions

once the plugin is installed, go to // packages/app/src/components/catalog/EntityPage.tsx and add below lines of code:
// In packages/app/src/components/catalog/EntityPage.tsx
    ```bash
    import {
    EntityGithubActionsContent,
    isGithubActionsAvailable,
    } from '@backstage/plugin-github-actions';

    // You can add the tab to any number of pages, the service page is shown as an
    // example here
    const serviceEntityPage = (
    <EntityLayout>
        {/* other tabs... */}
        <EntityLayout.Route path="/github-actions" title="GitHub Actions">
        <EntityGithubActionsContent />
        </EntityLayout.Route>
 
Restart your backstage application.
While I encountered errors and faced time constraints that hindered running my CI/CD pipeline directly from Backstage, the original intention was to initiate the pipeline using GitHub Actions integrated into Backstage.
 
Deploy the Container
I deployed the container on my localhost by initiating the GitHub workflow. Below is a snapshot captured from my Docker CLI:
 
Figure 5: Docker CLI
 
Startup script
This script is intendent to install backstage locally.
    ```bash
    startup.ps1
    # Install npx globally
    npm install -g npx

    # Install Backstage CLI
    Write-Host "Installing Backstage CLI..."
    npx @backstage/create-app@latest

    # Create a new Backstage app
    Write-Host "Creating a new Backstage app..."
    cd my-backstage-app

    # Start Backstage
    Write-Host "Starting Backstage..."
    yarn dev

How to Run the Script:
1.	Save the script as startup.ps1.
2.	Open a PowerShell terminal as an administrator.
    ```bash
    .\startup.ps1
 
## Deployment on AWS
To deploy your application on AWS, you can utilize Elastic Container Service (ECS) and set up an Application Load Balancer (ALB) for traffic distribution. Backstage seamlessly integrates with AWS for streamlined deployment.

•	ECS Cluster: Create an ECS cluster to manage your containers.
•	Docker Image Repository: Build and push your Docker image to a container registry like Amazon ECR (Elastic Container Registry).
•	ECS Task Definition: Define an ECS task that specifies your Docker container and its configurations.
•	ECS Service: Set up an ECS service to deploy and manage your tasks within the cluster.
•	ALB Setup: Optionally, configure an Application Load Balancer (ALB) to evenly distribute incoming traffic to your ECS service.
•	Backstage Integration: Leverage Backstage's AWS integration capabilities to streamline and automate the deployment process.
 
Challenges Encountered and Design Decisions
1.	 Installing Backstage in Different Environments:
Challenge: Initially started the Backstage installation on a production node, and later transitioning to a development node for local testing posed compatibility issues.

Resolution: Adjusted the installation process for a smoother transition between production and development nodes, ensuring compatibility and seamless local testing.

2.	 GitHub Actions Setup:
Challenge: Faced difficulties while setting up GitHub Actions for CI/CD. Although the pipeline could be run locally, GitHub Actions posed challenges.

Ongoing Resolution: Currently addressing the GitHub Actions setup challenges. While the pipeline runs locally, additional time and effort are needed for a more robust configuration.

3.	 Choosing Node.js for Application:
Design Decision: Given no specific technology mentioned, opted for Node.js due to its simplicity, wide adoption, and suitability for creating lightweight applications like "Hello World".
Rationale: Node.js provides a fast, event-driven, and scalable runtime, making it a pragmatic choice for a straightforward application, aligning with the task's simplicity.

4.	 Base Image for Docker Container:
Design Decision: Leveraged existing open-source Docker images as the base for the container, optimizing for efficiency and reliability.

Rationale: Using established base images ensures a secure and well-maintained foundation for the container, reducing potential vulnerabilities and improving overall container stability.

5.	Backstage Adoption Experience:
Achievement: Grasped knowledge quickly and efficiently while using Backstage for the first time.
Impact: Despite being a first-time user, successfully integrated Backstage into the development pipeline, showcasing adaptability and the ability to quickly comprehend and implement new tools.
