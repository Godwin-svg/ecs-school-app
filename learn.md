# Learning Documentation - ECS School App Project

## Overview

This document outlines the steps taken to set up and manage this project with Git and GitHub.

---

## Step 1: Creating GitHub Repository

1. **Created a new repository on GitHub**
   - Repository Name: `ecs-school-app`
   - Account: `Godwin-svg`
   - URL: `https://github.com/Godwin-svg/ecs-school-app`
   - Created without initializing README, .gitignore, or license

---

## Step 2: Setting Up Remote URL

### Check Current Remote

```bash
git remote -v
```

This shows which remote repository your local Git is connected to.

### Change Remote URL

Since the project was originally connected to `seunayolu/mini-schoolapp`, we needed to change it to our own repository:

```bash
git remote set-url origin git@github.com:Godwin-svg/ecs-school-app.git
```

### Verify Remote Change

```bash
git remote -v
```

Output should show:

```
origin  git@github.com:Godwin-svg/ecs-school-app.git (fetch)
origin  git@github.com:Godwin-svg/ecs-school-app.git (push)
```

---

## Step 3: Branch Management

### Viewing All Branches

```bash
git branch -a
```

This shows:

- Local branches
- Remote branches
- Current branch (marked with `*`)

### Creating New Branches

#### Create Backend Branch

```bash
git checkout -b backend
```

This creates a new branch called `backend` and switches to it.

#### Create Frontend Branch

```bash
git checkout -b frontend
```

This creates a new branch called `frontend` and switches to it.

### Switching Between Branches

```bash
git checkout main        # Switch to main branch
git checkout backend     # Switch to backend branch
git checkout frontend    # Switch to frontend branch
```

---

## Step 4: Pushing Branches to GitHub

### Push Main Branch

```bash
git push -u origin main
```

The `-u` flag sets up tracking between local and remote branch.

### Push Backend Branch

```bash
git checkout backend
git push -u origin backend
```

### Push Frontend Branch

```bash
git checkout frontend
git push -u origin frontend
```

---

## Step 5: Organizing Branch Content (Optional)

### Remove Frontend Folder from Backend Branch

```bash
git checkout backend
git rm -r frontend/
git commit -m "Remove frontend folder from backend branch"
git push origin backend
```

### Remove Backend Folder from Frontend Branch

```bash
git checkout frontend
git rm -r backend/
git commit -m "Remove backend folder from frontend branch"
git push origin frontend
```

### Keep Both Folders in Main Branch

The main branch retains both `backend/` and `frontend/` folders.

---

## Step 6: Moving Folder Contents to Root

### Moving Frontend Contents to Root (Frontend Branch)

To have frontend files directly at the root level instead of in a `frontend/` folder:

```bash
git checkout frontend
mv frontend/* ./
mv frontend/.* ./ 2>/dev/null || true
rm -rf frontend/
git add .
git commit -m "Move frontend contents to root directory"
git push origin frontend
```

**Result:** Frontend branch now has:

- package.json
- postcss.config.js
- tailwind.config.js
- src/
- public/
- (no frontend/ folder)

### Moving Backend Contents to Root (Backend Branch)

Similar process for backend:

```bash
git checkout backend
mv backend/* ./
rm -rf backend/
git add .
git commit -m "Move backend contents to root directory"
git push origin backend
```

**Result:** Backend branch now has:

- index.js
- package.json
- config/
- controllers/
- models/
- routes/
- (no backend/ folder)

### Branch Structure Summary

- **Main Branch:** Contains both `backend/` and `frontend/` folders
- **Frontend Branch:** Frontend files at root level, backend folder deleted
- **Backend Branch:** Backend files at root level, frontend folder deleted

---

## Step 7: Common Errors and Troubleshooting

### Error: `git mv` with Empty Directory (node_modules)

**Error Message:**

```
fatal: source directory is empty, source=frontend/node_modules, destination=node_modules
```

**Cause:**

- `node_modules` is not tracked by Git (it's in `.gitignore`)
- `git mv` only works with tracked files

**Solution:**
Use regular `mv` command instead of `git mv`:

```bash
mv frontend/* ./
```

---

### Error: Package Lock File Out of Sync (AWS Amplify Build)

**Error Message:**

```
npm error `npm ci` can only install packages when your package.json and
package-lock.json or npm-shrinkwrap.json are in sync.

npm error Invalid: lock file's typescript@5.9.3 does not satisfy typescript@4.9.5
```

**Cause:**

- `package.json` specifies TypeScript 4.9.5
- `package-lock.json` has TypeScript 5.9.3
- AWS Amplify uses `npm ci` which requires exact sync

**Solution:**

1. **Switch to frontend branch:**

```bash
git checkout frontend
```

2. **Regenerate package-lock.json:**

```bash
npm install
```

3. **Commit and push updated lock file:**

```bash
git add package-lock.json
git commit -m "Update package-lock.json to sync with package.json"
git push origin frontend
```

4. **AWS Amplify will automatically rebuild**

**Prevention:**

- Always run `npm install` after modifying `package.json`
- Commit both `package.json` and `package-lock.json` together
- Never manually edit `package-lock.json`

---

## Step 8: AWS Security Groups Configuration

Security groups act as virtual firewalls to control inbound and outbound traffic for AWS resources.

### Security Group 1: Application Load Balancer (ALB)

**Name:** `schoolApp-alb-sg`

**Purpose:** Controls traffic to the Application Load Balancer

**Inbound Rules:**

- **HTTP (Port 80):** Allow from `0.0.0.0/0` (anywhere on the internet)
- **HTTPS (Port 443):** Allow from `0.0.0.0/0` (anywhere on the internet)

**Outbound Rules:**

- **All traffic:** Allow to `0.0.0.0/0` (all destinations)

**Description:** This security group allows public access to the load balancer via HTTP and HTTPS, enabling users to access the application from the internet.

---

### Security Group 2: ECS Services

**Name:** `schoolApp-ecs-sg`

**Purpose:** Controls traffic to ECS containers running the backend application

**Inbound Rules:**

- **Custom TCP (Port 3000):** Allow from `schoolApp-alb-sg` (only from the ALB security group)

**Outbound Rules:**

- **All traffic:** Allow to `0.0.0.0/0` (all destinations)

**Description:** This security group ensures that only the Application Load Balancer can communicate with the ECS containers on port 3000. This provides an additional layer of security by preventing direct access to the containers from the internet.

---

### Security Group 3: RDS Database

**Name:** `schoolApp-rds-sg`

**Purpose:** Controls traffic to the RDS MySQL database

**Inbound Rules:**

- **MySQL/Aurora (Port 3306):** Allow from `schoolApp-ecs-sg` (only from the ECS security group)

**Outbound Rules:**

- **All traffic:** Allow to `0.0.0.0/0` (all destinations)

**Description:** This security group restricts database access to only the ECS containers. No other resources can directly connect to the database, ensuring data security.

---

### Security Groups Architecture Flow

```
Internet (0.0.0.0/0)
       ↓
       ↓ (HTTP/HTTPS: 80, 443)
       ↓
[schoolApp-alb-sg] - Application Load Balancer
       ↓
       ↓ (TCP: 3000)
       ↓
[schoolApp-ecs-sg] - ECS Containers (Backend)
       ↓
       ↓ (MySQL: 3306)
       ↓
[schoolApp-rds-sg] - RDS Database
```

**Key Security Benefits:**

1. **Layered Defense:** Each layer only accepts traffic from the previous layer
2. **Principle of Least Privilege:** Each resource only has the minimum required access
3. **No Direct Database Access:** Database is isolated and can only be accessed through the application
4. **Public Access Control:** Only the ALB is publicly accessible

---

## Step 9: Containerizing the Backend with Docker

### Creating the Dockerfile

Created a `Dockerfile` to containerize the Node.js backend application using Node.js 23 Alpine base image.

**Dockerfile:**

```dockerfile
# Use Node.js 23 Alpine as base image
FROM node:23-alpine

# Set working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port 3000
EXPOSE 3000

# Start the application
CMD ["node", "index.js"]
```

**Dockerfile Breakdown:**

1. **Base Image:** `node:23-alpine` - Lightweight Alpine Linux with Node.js 23
2. **Working Directory:** Sets `/app` as the working directory inside the container
3. **Copy Dependencies:** Copies `package.json` and `package-lock.json` first (for layer caching)
4. **Install Dependencies:** Uses `npm ci` for clean, production-only install
5. **Copy Code:** Copies the application source code
6. **Expose Port:** Exposes port 3000 (matches ECS security group configuration)
7. **Start Command:** Runs the application with `node index.js`

---

### Creating the .dockerignore File

Created a `.dockerignore` file to exclude unnecessary files from the Docker build context.

**.dockerignore:**

```
# Dependencies
node_modules/
npm-debug.log*

# Git files
.git/
.gitignore

# Docker files
Dockerfile
.dockerignore

# Documentation
README.md
learn.md

# Environment files
.env
.env.local

# IDE and editor files
.vscode/
.idea/
*.swp
*.swo
*~

# OS files
.DS_Store
Thumbs.db

# Test files
coverage/
*.test.js
*.spec.js

# Build artifacts
dist/
build/
```

**How .dockerignore Works:**

Similar to `.gitignore`, but for Docker builds. When running `docker build`, Docker sends files to the build context. The `.dockerignore` file tells Docker which files to exclude.

**When this line runs in Dockerfile:**

```dockerfile
COPY . .
```

**Without .dockerignore:** Copies EVERYTHING (including node_modules, .git, Dockerfile, etc.)

**With .dockerignore:** Copies ONLY necessary application files

**Benefits:**

1. **Smaller Images:** Excludes unnecessary files, reducing image size
2. **Faster Builds:** Less data to transfer to Docker daemon
3. **Security:** Prevents sensitive files (.env, .git) from being in the image
4. **Clean Installation:** node_modules is reinstalled fresh via `npm ci` in the container
5. **Consistency:** Ensures production environment matches the Dockerfile specifications

**Files Included in Docker Image:**

- `index.js`
- `package.json`, `package-lock.json`
- `config/`, `controllers/`, `models/`, `routes/`
- Application source code

**Files Excluded from Docker Image:**

- `node_modules/` (reinstalled fresh inside container)
- Git-related files
- Documentation
- Development and IDE files

---

## Step 10: AWS Systems Manager Parameter Store Configuration

AWS Systems Manager Parameter Store provides secure storage for configuration data and secrets. The backend application retrieves database connection details from Parameter Store.

### Parameter Store Setup

Created three parameters in AWS Systems Manager Parameter Store to store RDS database connection information securely.

### Parameters Created

**1. Database User Parameter**

- **Name:** `/schoolapp/dev/db_user`
- **Type:** SecureString
- **Description:** Database username for the school app
- **Value:** Retrieved from RDS database configuration

**2. Database Name Parameter**

- **Name:** `/schoolapp/dev/db_name`
- **Type:** SecureString
- **Description:** Database name for the school app
- **Value:** Retrieved from RDS database configuration

**3. Database Host Parameter**

- **Name:** `/schoolapp/dev/db_host`
- **Type:** SecureString
- **Description:** RDS database endpoint/host
- **Value:** Retrieved from RDS database endpoint

### Parameter Store Configuration in aws.js

The `config/aws.js` file retrieves these parameters to establish database connections. Here's what happens step by step:

**Step 1: Retrieve Configuration from Parameter Store**

```javascript
// Retrieve database host, name, and user from Parameter Store
const hostParam = await ssmClient.send(
  new GetParameterCommand({
    Name: process.env.PARAM_HOST_PATH,
    WithDecryption: true,
  }),
);
const dbNameParam = await ssmClient.send(
  new GetParameterCommand({
    Name: process.env.PARAM_DB_NAME_PATH,
    WithDecryption: true,
  }),
);
const userParam = await ssmClient.send(
  new GetParameterCommand({
    Name: process.env.PARAM_USER_PATH,
    WithDecryption: true,
  }),
);

const host = hostParam.Parameter.Value; // e.g., "schoolapp-db.us-east-1.rds.amazonaws.com"
const dbName = dbNameParam.Parameter.Value; // e.g., "schoolapp_db"
const user = userParam.Parameter.Value; // e.g., "admin"
```

**Step 2: What Does It Do With These Values?**

After retrieving these values, the application uses them to create a connection to the MySQL database:

```javascript
// Create a MySQL connection using the retrieved values
const connection = await mysql.createConnection({
  host: host, // Where the database is located
  user: user, // Username to login to the database
  password: password, // Password from Secrets Manager
  database: dbName, // Which database to connect to
});

return connection; // Return the connection for use in the application
```

**Step 3: Why Retrieve from Parameter Store?**

**Without Parameter Store (Hardcoded - BAD):**

```javascript
const connection = await mysql.createConnection({
  host: "schoolapp-db.us-east-1.rds.amazonaws.com", // Hardcoded
  user: "admin", // Hardcoded
  password: "mypassword123", // Hardcoded and exposed!
  database: "schoolapp_db", // Hardcoded
});
```

Problems:

- ❌ Configuration is hardcoded in the code
- ❌ Need to rebuild and redeploy to change database
- ❌ Cannot have separate dev/staging/prod databases
- ❌ Secrets are exposed in the code

**With Parameter Store (Dynamic - GOOD):**

```javascript
const host = await getFromParameterStore("/schoolapp/dev/db_host");
const user = await getFromParameterStore("/schoolapp/dev/db_user");
const dbName = await getFromParameterStore("/schoolapp/dev/db_name");
```

Benefits:

- ✅ Configuration is stored securely in AWS
- ✅ Can change database without code changes
- ✅ Can have different values for dev/staging/prod
- ✅ No secrets in the source code
- ✅ Easy to update (just change parameter value in AWS)

**Real-World Use Case:**

When the ECS container starts, it:

1. Retrieves host, user, and database name from Parameter Store
2. Retrieves password from Secrets Manager
3. Creates a MySQL connection using these values
4. Uses this connection to run queries like:
   - `SELECT * FROM users`
   - `INSERT INTO registrations...`
   - etc.

**Example: User Registration Flow**

```
User submits form → Backend API (ECS) → Needs to save to database
                                       ↓
                           Gets connection from aws.js
                                       ↓
                           Uses retrieved values to connect
                                       ↓
                           Runs: INSERT INTO users (name, email)...
```

### Environment Variables Required

The application expects these environment variables to be set:

- `PARAM_HOST_PATH=/schoolapp/dev/db_host`
- `PARAM_DB_NAME_PATH=/schoolapp/dev/db_name`
- `PARAM_USER_PATH=/schoolapp/dev/db_user`
- `AWS_REGION=us-east-1`

---

### Visual Flow Diagram

```
┌─────────────────┐
│  User Browser   │  1. User fills form and clicks "Register"
│  (Frontend)     │
└────────┬────────┘
         │ HTTP POST: /api/register
         │ Body: { name, email, phone, program }
         ↓
┌─────────────────┐
│   index.js      │  2. Express receives request
│  (Main Server)  │
└────────┬────────┘
         │ Routes to /api
         ↓
┌─────────────────┐
│ userRoutes.js   │  3. Matches /register route
│   (Routes)      │
└────────┬────────┘
         │ Calls registerUser()
         ↓
┌──────────────────┐
│userController.js │  4. Validates data (email, phone, etc.)
│  (Controller)    │
└────────┬─────────┘
         │ Calls createUser()
         ↓
┌─────────────────┐
│ userModel.js    │  5. Needs database connection
│   (Model)       │     Calls getDbConnection()
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│    aws.js       │  6. Retrieves credentials from AWS:
│ (AWS Config)    │     - Password from Secrets Manager
└────────┬────────┘     - Host/User/DB from Parameter Store
         │              Creates MySQL connection
         │
         ↓
┌─────────────────────────────┐
│      AWS Services           │
│                             │
│  ┌────────────────────┐    │
│  │ Secrets Manager    │    │  Password
│  └────────────────────┘    │
│                             │
│  ┌────────────────────┐    │
│  │ Parameter Store    │    │  Host, User, DB Name
│  └────────────────────┘    │
│                             │
│  ┌────────────────────┐    │
│  │  RDS MySQL         │    │  Stores user data
│  │  Database          │    │
│  └────────────────────┘    │
└─────────────────────────────┘
         │
         │ Connection established!
         ↓
┌─────────────────┐
│  userModel.js   │  7. Executes SQL INSERT query
└────────┬────────┘
         │ Returns success
         ↓
┌──────────────────┐
│userController.js │  8. Sends response to frontend
└────────┬─────────┘
         │ Status 201: User created
         ↓
┌─────────────────┐
│  User Browser   │  9. Shows "Registration Successful!"
└─────────────────┘
```

---

## Step 11: How the Codebase Builds and Runs the Application

### Understanding the Complete Picture

This codebase **IS** the application. All the code files work together to create the backend API that users interact with.

### The Code Files and Their Roles

**Backend Application Files:**

- **[index.js](index.js)** - Starts the Express server and listens on port 3000
- **[routes/userRoutes.js](routes/userRoutes.js)** - Defines API endpoints (e.g., `/api/register`)
- **[controllers/userController.js](controllers/userController.js)** - Validates user input and handles business logic
- **[models/userModel.js](models/userModel.js)** - Runs database queries (INSERT, SELECT, etc.)
- **[config/aws.js](config/aws.js)** - Connects to AWS services and RDS database

**Build and Deployment Files:**

- **[Dockerfile](Dockerfile)** - Packages the application into a Docker container image
- **[.dockerignore](.dockerignore)** - Excludes unnecessary files from the Docker image
- **[package.json](package.json)** - Lists all Node.js dependencies
- **[package-lock.json](package-lock.json)** - Locks exact versions of dependencies

### The Build Process Flow

```
1. Write Code
   ├── index.js
   ├── routes/
   ├── controllers/
   ├── models/
   └── config/

2. Dockerfile Builds Image
   └── docker build -t schoolapp-backend .
       ├── Installs Node.js 23 Alpine
       ├── Copies package.json
       ├── Runs npm ci (installs dependencies)
       ├── Copies application code
       └── Creates container image

3. Push to AWS ECR (Elastic Container Registry)
   └── docker push <ecr-url>/schoolapp-backend:latest

4. ECS Pulls and Runs Image
   └── Creates containers from the image
       ├── Sets environment variables
       ├── Connects to VPC and security groups
       └── Starts the application (node index.js)

5. ALB Routes Traffic
   └── User requests → ALB → ECS containers

6. Application Runs
   └── Containers execute your code:
       - Express server listens on port 3000
       - Handles /api/register requests
       - Connects to RDS via aws.js
       - Saves data to database
```

### From Code to Running Application

**Step 1: Local Development**

```bash
# Your code files exist on your computer
/Users/innocentgodwin/Desktop/mini-schoolapp/
├── index.js
├── controllers/
├── models/
└── ...
```

**Step 2: Docker Containerization**

```bash
# Dockerfile packages everything into an image
docker build -t schoolapp-backend .

# Image contains:
# - Node.js runtime
# - Your code files
# - All dependencies (express, mysql2, aws-sdk, etc.)
# - Ready to run anywhere
```

**Step 3: Push to AWS ECR**

```bash
# Image is stored in AWS
aws ecr get-login-password | docker login --username AWS --password-stdin <ecr-url>
docker tag schoolapp-backend:latest <ecr-url>/schoolapp-backend:latest
docker push <ecr-url>/schoolapp-backend:latest
```

**Step 4: ECS Deployment**

- ECS Task Definition points to the ECR image
- ECS Service creates containers from that image
- Containers run in your VPC with security groups
- Environment variables are injected (PARAM_HOST_PATH, DB_SECRET_ARN, etc.)

**Step 5: Application Starts**

```bash
# Inside the container, this command runs:
node index.js

# Output:
# Server running on port 3000
# Database connection established
```

**Step 6: Application Handles Requests**

```
User → ALB → ECS Container → Your Code Executes:
                              ├── index.js receives request
                              ├── userRoutes.js routes to controller
                              ├── userController.js validates data
                              ├── userModel.js queries database
                              ├── aws.js gets DB credentials from AWS
                              └── Response sent back to user
```

### How User Registration Works with This Codebase

**User Action: Fills form and clicks "Register"**

1. **Frontend sends POST request:**

   ```
   POST https://api.schoolapp.com/api/register
   Body: { name: "John", email: "john@example.com", ... }
   ```

2. **ALB receives request** → Forwards to ECS container

3. **Container runs your code:**
   - `index.js` receives the HTTP request
   - `userRoutes.js` matches `/register` route
   - `userController.js` validates the data
   - `userModel.js` needs to save to database
   - `aws.js` is called to get database connection
   - `aws.js` retrieves credentials from Parameter Store & Secrets Manager
   - MySQL connection is created to RDS
   - SQL INSERT query runs: `INSERT INTO users (id, name, email, ...) VALUES (...)`
   - Success response returned

4. **User sees:** "Registration Successful!"

### The Complete Lifecycle

```
┌──────────────────────────────────────────────────┐
│  DEVELOPMENT PHASE                               │
├──────────────────────────────────────────────────┤
│  1. Write code (index.js, controllers, etc.)    │
│  2. Test locally                                 │
│  3. Commit to GitHub (backend branch)            │
└──────────────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────────┐
│  BUILD PHASE                                     │
├──────────────────────────────────────────────────┤
│  4. Dockerfile builds container image            │
│  5. Image pushed to AWS ECR                      │
└──────────────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────────┐
│  DEPLOYMENT PHASE                                │
├──────────────────────────────────────────────────┤
│  6. ECS pulls image from ECR                     │
│  7. ECS creates containers                       │
│  8. Containers run in VPC with security groups   │
│  9. ALB configured to route traffic              │
└──────────────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────────────┐
│  RUNTIME PHASE                                   │
├──────────────────────────────────────────────────┤
│  10. Application starts (node index.js)          │
│  11. Connects to AWS services                    │
│  12. Ready to handle user requests               │
│  13. Users register → Data saved to RDS          │
└──────────────────────────────────────────────────┘
```

### Key Takeaways

1. **Your code IS the application** - The files you write contain all the logic
2. **Dockerfile packages it** - Makes it portable and consistent
3. **AWS runs it** - ECS provides the infrastructure
4. **Security groups protect it** - Layered defense as shown in Step 8
5. **Parameter Store secures credentials** - No secrets in code
6. **Everything connects** - From user browser to database and back

**The codebase you've built creates a complete, production-ready backend API that handles user registration, validates data, stores it securely in RDS, and responds to the frontend - all running in AWS infrastructure.**

---

## Step 12: Pushing Docker Image to AWS ECR

AWS Elastic Container Registry (ECR) is a fully managed Docker container registry that stores Docker images securely in AWS, making them accessible to ECS for deployment.

### Prerequisites

- Docker installed locally
- AWS CLI configured with appropriate credentials
- IAM permissions for ECR operations

---

### Step 1: Create ECR Repository

Created a repository in AWS ECR to store the backend Docker images.

**Via AWS Console:**

1. Navigate to Amazon ECR in AWS Console
2. Click "Create repository"
3. Repository name: `schoolapp-backend`
4. Leave other settings as default
5. Click "Create repository"

**Repository URI:**

```
058264237826.dkr.ecr.us-east-1.amazonaws.com/schoolapp-backend
```

---

### Step 2: Build Docker Image with ECR Tag

Built the Docker image using the ECR repository URI as the tag. This ensures the image is properly named for pushing to ECR.

```bash
docker build -t 058264237826.dkr.ecr.us-east-1.amazonaws.com/schoolapp-backend:1.0 .
```

**What this does:**

- `-t` flag specifies the tag/name for the image
- Format: `<ECR_URI>/<repository-name>:<version>`
- Version `1.0` tags this as the first release

---

### Step 3: Initial Push Attempt (Error Encountered)

Attempted to push the image to ECR:

```bash
docker push 058264237826.dkr.ecr.us-east-1.amazonaws.com/schoolapp-backend:1.0
```

**Error Received:**

```
unknown: unexpected status from HEAD request to https://058264237826.dkr.ecr.us-east-1.amazonaws.com/v2/schoolapp-backend/blobs/sha256:...: 403 Forbidden
```

**Why this happened:**

- Docker was not authenticated with ECR
- ECR requires authentication tokens that expire after 12 hours
- The current terminal session didn't have valid ECR credentials

---

### Step 4: Set AWS Profile

Set the AWS CLI profile to use the correct credentials:

```bash
export AWS_PROFILE=dev2
```

**What this does:**

- Tells AWS CLI to use the `dev2` profile credentials
- The `dev2` profile contains IAM user access keys with ECR permissions
- This environment variable affects all subsequent AWS CLI commands in this terminal session

**Verify profile:**

```bash
aws configure list
```

**Output:**

```
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                     dev2              env    ['AWS_PROFILE', 'AWS_DEFAULT_PROFILE']
access_key     ****************ACX3 shared-credentials-file
secret_key     ****************2tio shared-credentials-file
    region                us-east-1      config-file    ~/.aws/config
```

---

### Step 5: Authenticate Docker with ECR

Logged Docker into the ECR registry using AWS CLI:

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 058264237826.dkr.ecr.us-east-1.amazonaws.com
```

**Command Breakdown:**

1. **`aws ecr get-login-password --region us-east-1`**
   - Retrieves a temporary authentication token from ECR
   - Token is valid for 12 hours
   - Uses credentials from the `dev2` profile

2. **`|` (pipe)**
   - Passes the password output to the next command

3. **`docker login --username AWS --password-stdin 058264237826.dkr.ecr.us-east-1.amazonaws.com`**
   - `--username AWS` - ECR always uses "AWS" as the username
   - `--password-stdin` - Reads password from the pipe (more secure than typing)
   - Authenticates Docker CLI with the ECR registry

**Success Output:**

```
Login Succeeded
```

**What this achieves:**

- Docker now has temporary credentials to push/pull images from this ECR repository
- Credentials are stored in `~/.docker/config.json`
- Must re-run this command when the token expires (after 12 hours)

---

### Step 6: Push Docker Image to ECR (Success)

With Docker authenticated, pushed the image to ECR:

```bash
docker push 058264237826.dkr.ecr.us-east-1.amazonaws.com/schoolapp-backend:1.0
```

**Push Process:**

```
The push refers to repository [058264237826.dkr.ecr.us-east-1.amazonaws.com/schoolapp-backend]
a078b3ff0fd0: Pushed
36b1c0ff86b0: Pushed
58728aadfd5a: Pushed
70563f461ecc: Pushed
e6963947ece0: Pushed
6f6decffc46a: Pushed
e7617a7b6a5a: Pushed
baaf468aa51f: Pushed
d69d4d41cfe2: Pushed
1.0: digest: sha256:... size: 2213
```

**Success!** The Docker image is now stored in AWS ECR.

---

### What Happens After Push

1. **Image Layers Uploaded:**
   - Each line represents a layer in the Docker image
   - Layers are cached in ECR
   - Future pushes only upload changed layers (faster)

2. **Image Available for ECS:**
   - ECS can now pull this image to create containers
   - Image URI: `058264237826.dkr.ecr.us-east-1.amazonaws.com/schoolapp-backend:1.0`

3. **Next Steps:**
   - Create ECS Task Definition pointing to this image
   - Create ECS Service to run containers
   - Configure ALB to route traffic to containers

---

### Key Commands Summary

```bash
# 1. Set AWS profile
export AWS_PROFILE=dev2

# 2. Create ECR repository (AWS Console or CLI)
aws ecr create-repository --repository-name schoolapp-backend --region us-east-1

# 3. Build Docker image with ECR tag
docker build -t 058264237826.dkr.ecr.us-east-1.amazonaws.com/schoolapp-backend:1.0 .

# 4. Authenticate Docker with ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 058264237826.dkr.ecr.us-east-1.amazonaws.com

# 5. Push image to ECR
docker push 058264237826.dkr.ecr.us-east-1.amazonaws.com/schoolapp-backend:1.0
```

---

### Troubleshooting ECR Push Errors

**Error: 403 Forbidden**

```
unknown: unexpected status from HEAD request to https://...: 403 Forbidden
```

**Solutions:**

1. Ensure AWS CLI profile is set: `export AWS_PROFILE=dev2`
2. Re-authenticate with ECR: Run the login command again
3. Verify IAM permissions: User must have ECR push permissions

**Required IAM Permissions:**

- `ecr:GetAuthorizationToken`
- `ecr:BatchCheckLayerAvailability`
- `ecr:PutImage`
- `ecr:InitiateLayerUpload`
- `ecr:UploadLayerPart`
- `ecr:CompleteLayerUpload`

**Error: Repository does not exist**

```
repository schoolapp-backend not found
```

**Solution:**
Create the ECR repository first before pushing:

```bash
aws ecr create-repository --repository-name schoolapp-backend --region us-east-1
```

---

### ECR Image Management

**View Images in Repository:**

```bash
aws ecr list-images --repository-name schoolapp-backend --region us-east-1
```

**Delete Old Images:**

```bash
aws ecr batch-delete-image --repository-name schoolapp-backend --image-ids imageTag=1.0 --region us-east-1
```

**Pull Image Locally:**

```bash
docker pull 058264237826.dkr.ecr.us-east-1.amazonaws.com/schoolapp-backend:1.0
```

---

## Step 13: Creating ECS Cluster and Task Definition

This step covers creating an ECS cluster, configuring IAM policies and roles for secure AWS service access, and setting up the ECS task definition to run the containerized backend application.

---

### Step 1: Create ECS Cluster

Created an ECS cluster to host and manage the backend containers.

**Via AWS Console:**

1. Navigate to Amazon ECS
2. Click "Create Cluster"
3. Enter cluster name: `schoolapp-cluster`
4. Select infrastructure: AWS Fargate
5. Click "Create"

**Via AWS CLI:**

```bash
aws ecs create-cluster --cluster-name schoolapp-cluster --region us-east-1
```

**What is an ECS Cluster?**

- Logical grouping of ECS tasks and services
- Manages container orchestration
- Can run on Fargate (serverless) or EC2 instances
- In this project: Using **AWS Fargate** (no servers to manage)

---

### Step 2: Create IAM Policy for Backend Application

Created a custom IAM policy named `schoolapp-backend-policy` to grant the ECS tasks permissions to access Secrets Manager and Systems Manager Parameter Store.

**Via AWS Console:**

1. Navigate to IAM → Policies
2. Click "Create policy"
3. Select JSON tab
4. Paste the policy JSON (see below)
5. Click "Next: Tags"
6. Click "Next: Review"
7. Policy name: `schoolapp-backend-policy`
8. Click "Create policy"

**Policy JSON:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "SecretsManagerAccess",
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue",
        "secretsmanager:DescribeSecret"
      ],
      "Resource": "arn:aws:secretsmanager:us-east-1:058264237826:secret:rds!db-f188ccd3-92e2-4237-92c4-201f1b24855d-pPkUpp"
    },
    {
      "Sid": "SystemsManagerAccess",
      "Effect": "Allow",
      "Action": ["ssm:GetParameter", "ssm:GetParameters"],
      "Resource": [
        "arn:aws:ssm:us-east-1:058264237826:parameter/schoolapp/dev/db_host",
        "arn:aws:ssm:us-east-1:058264237826:parameter/schoolapp/dev/db_name",
        "arn:aws:ssm:us-east-1:058264237826:parameter/schoolapp/dev/db_user"
      ]
    }
  ]
}
```

**Policy Breakdown:**

**1. Secrets Manager Permissions:**

- `secretsmanager:GetSecretValue` - Retrieve database password
- `secretsmanager:DescribeSecret` - Get secret metadata

**2. Systems Manager Permissions:**

- `ssm:GetParameter` - Retrieve individual parameter
- `ssm:GetParameters` - Retrieve multiple parameters at once

**Why These Permissions?**

- The application code in [config/aws.js](config/aws.js) requires these permissions to:
  - Retrieve database password from Secrets Manager
  - Retrieve host, username, and database name from Parameter Store

---

### Step 3: Create IAM Role for ECS Tasks

Created an IAM role named `schoolapp-backend-role` that ECS tasks will assume to access AWS services.

**Via AWS Console:**

1. Navigate to IAM → Roles
2. Click "Create role"
3. Select trusted entity type: **AWS service**
4. Use case: **Elastic Container Service**
5. Select: **Elastic Container Service Task**
   - Description: "Allows ECS tasks to call AWS services on your behalf"
6. Click "Next"
7. Attach permissions policies:
   - Select `schoolapp-backend-policy` (created in Step 2)
8. Click "Next"
9. Role name: `schoolapp-backend-role`
10. Click "Create role"

**What is a Task Role?**

- IAM role that ECS tasks assume when running
- Grants permissions to the application code running inside containers
- Different from **Task Execution Role** (used by ECS agent to pull images, write logs)

**Trust Relationship:**
The role automatically gets this trust policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs-tasks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

This allows ECS tasks to assume this role.

---

### Step 4: Create ECS Task Definition

Created a task definition that specifies how ECS should run the backend container.

**Via AWS Console:**

**1. Navigate to Task Definitions:**

- Amazon ECS → Task Definitions
- Click "Create new task definition"

**2. Basic Configuration:**

- Task definition family: `schoolapp-backend-task`
- Launch type: **AWS Fargate**
- Operating system: **Linux/ARM64**
  - _(Selected ARM64 because building on Mac with Apple Silicon)_
- Task role: `schoolapp-backend-role` _(from Step 3)_
- Task execution role: `ecsTaskExecutionRole` _(default, auto-created by AWS)_

**3. Container Configuration:**

**Container Name:** `schoolapp-backend-container`

**Image URI:**

```
058264237826.dkr.ecr.us-east-1.amazonaws.com/schoolapp-backend:1.0
```

**Port Mappings:**

- Container port: `3000`
- Protocol: TCP
- App protocol: HTTP
- Name: `http-3000`

**4. Environment Variables:**

Added the following environment variables required by [config/aws.js](config/aws.js):

| Key                  | Value                                                                                                     |
| -------------------- | --------------------------------------------------------------------------------------------------------- |
| `AWS_REGION`         | `us-east-1`                                                                                               |
| `DB_SECRET_ARN`      | `arn:aws:secretsmanager:us-east-1:058264237826:secret:rds!db-f188ccd3-92e2-4237-92c4-201f1b24855d-pPkUpp` |
| `PARAM_HOST_PATH`    | `/schoolapp/dev/db_host`                                                                                  |
| `PARAM_DB_NAME_PATH` | `/schoolapp/dev/db_name`                                                                                  |
| `PARAM_USER_PATH`    | `/schoolapp/dev/db_user`                                                                                  |

**Important Note on Parameter Store Paths:**

- Use parameter **names** (e.g., `/schoolapp/dev/db_host`)
- NOT full ARNs (e.g., `arn:aws:ssm:...`)
- The AWS SDK automatically constructs the full ARN from the region and name

**5. Resource Allocation:**

- CPU: 0.5 vCPU (512 units)
- Memory: 1 GB (1024 MiB)

**6. Logging Configuration:**

- Log driver: awslogs
- Log group: `/ecs/schoolapp-backend` (auto-created)
- Region: us-east-1
- Stream prefix: ecs

**7. Click "Create"**

---

### Task Definition Breakdown

**What is a Task Definition?**

- Blueprint for running containers in ECS
- Similar to a Docker Compose file
- Defines: images, resources, environment variables, networking

**Task Role vs Task Execution Role:**

| Role Type               | Purpose                    | Used By               | Example Permissions                              |
| ----------------------- | -------------------------- | --------------------- | ------------------------------------------------ |
| **Task Role**           | Application permissions    | Your application code | Access Secrets Manager, Parameter Store, S3, RDS |
| **Task Execution Role** | Infrastructure permissions | ECS agent             | Pull ECR images, write CloudWatch logs           |

**In this setup:**

- **Task Role:** `schoolapp-backend-role` (custom, with our policy)
- **Task Execution Role:** `ecsTaskExecutionRole` (AWS-managed default)

---

### Why ARM64?

**Selected Linux/ARM64 because:**

- Building Docker image on Mac with Apple Silicon (M1/M2/M3)
- Docker builds ARM64 images by default on Apple Silicon
- Must match container architecture to avoid compatibility issues

**If using x86/AMD64:**

```bash
# Build for x86 architecture
docker buildx build --platform linux/amd64 -t <ecr-uri> .
```

---

### Environment Variables Explained

These environment variables connect the application to AWS services:

**1. AWS_REGION**

- Specifies AWS region for SDK clients
- Required by `SecretsManagerClient` and `SSMClient` in [aws.js](config/aws.js)

**2. DB_SECRET_ARN**

- Full ARN of the Secrets Manager secret containing database password
- Used to retrieve password: `process.env.DB_SECRET_ARN`

**3. PARAM_HOST_PATH**

- Parameter Store path for RDS endpoint
- Retrieved: `ssmClient.send(new GetParameterCommand({ Name: process.env.PARAM_HOST_PATH }))`

**4. PARAM_DB_NAME_PATH**

- Parameter Store path for database name
- Retrieved: `ssmClient.send(new GetParameterCommand({ Name: process.env.PARAM_DB_NAME_PATH }))`

**5. PARAM_USER_PATH**

- Parameter Store path for database username
- Retrieved: `ssmClient.send(new GetParameterCommand({ Name: process.env.PARAM_USER_PATH }))`

---

### Verification

**View Task Definition:**

```bash
aws ecs describe-task-definition --task-definition schoolapp-backend-task --region us-east-1
```

**List Task Definitions:**

```bash
aws ecs list-task-definitions --region us-east-1
```

**View IAM Role:**

```bash
aws iam get-role --role-name schoolapp-backend-role
```

**View Attached Policies:**

```bash
aws iam list-attached-role-policies --role-name schoolapp-backend-role
```

---

### What Happens When Task Runs

```
1. ECS pulls container image from ECR
   └── Uses Task Execution Role

2. Container starts with environment variables

3. Application starts (node index.js)

4. Application needs database credentials
   └── Calls getDbConnection() in aws.js

5. aws.js uses Task Role to:
   ├── Call Secrets Manager (get password)
   └── Call Parameter Store (get host, user, dbName)

6. MySQL connection established

7. Application ready to handle requests on port 3000
```

---

## Step 14: Creating ECS Service with Load Balancer

This step covers creating an ECS Service to run and maintain the backend containers, and configuring an Application Load Balancer to route HTTPS traffic to the containers.

---

### What is an ECS Service?

An ECS Service ensures that:

- A specified number of tasks (containers) are always running
- Failed tasks are automatically replaced
- Tasks are distributed across availability zones
- Traffic is load-balanced across healthy tasks
- Deployments are managed with zero downtime

---

### Step 1: Create ECS Service

Created an ECS Service in the `schoolapp-cluster` to run the backend application.

**Via AWS Console:**

**1. Navigate to ECS Cluster:**

- Amazon ECS → Clusters → `schoolapp-cluster`
- Click "Create" under Services

**2. Environment Configuration:**

- **Compute options:** Launch type
- **Launch type:** **Fargate**
  - Uses serverless compute (no EC2 instances to manage)

**3. Deployment Configuration:**

- **Application type:** Service
- **Task definition:** `schoolapp-backend-task` (latest revision)
- **Service name:** `schoolapp-backend-service`
- **Service type:** **Replica**
  - Maintains a specified number of identical tasks
- **Desired tasks:** **2**
  - Runs 2 container instances for high availability
  - Distributes across multiple availability zones

**4. Deployment Options:**

- **Deployment type:** Rolling update
  - Gradually replaces old tasks with new ones
  - Maintains service availability during deployments
- **Min running tasks:** 100%
  - Ensures at least 100% of desired tasks run during deployment
- **Max running tasks:** 200%
  - Allows up to 200% during deployment (old + new tasks)

**Why Rolling Update?**

- Zero downtime deployments
- New tasks are started before old tasks are stopped
- If new tasks fail health checks, deployment stops automatically

---

### Step 2: Configure Networking

**VPC Configuration:**

- **VPC:** Selected your VPC (same VPC as RDS and ALB)
- **Subnets:** Selected **2 private subnets**
  - Tasks run in private subnets (not directly accessible from internet)
  - ALB in public subnets routes traffic to tasks

**Why Private Subnets?**

- ✅ Enhanced security (containers not directly exposed to internet)
- ✅ Only ALB can reach containers
- ✅ Outbound internet access via NAT Gateway (for pulling images, accessing AWS services)

**Security Group:**

- **Selected existing security group:** `schoolApp-ecs-sg`
- Allows inbound traffic on port 3000 from `schoolApp-alb-sg`
- Allows outbound traffic to all destinations (for database, AWS services)

**Public IP:**

- **Auto-assign public IP:** Disabled (not needed in private subnets with NAT Gateway)

---

### Step 3: Configure Load Balancer

**Load Balancer Type:**

- **Application Load Balancer (ALB)**

**Load Balancer Configuration:**

- **Port:** **443** (HTTPS)
- **Target container:** `schoolapp-backend-container:3000`
  - Forwards traffic from ALB port 443 to container port 3000

**Target Group:**

- **Target type:** IP (Fargate uses IP-based targeting)
- **Protocol:** HTTP
- **Port:** 3000
- **Health check path:** `/health` (or `/` if no dedicated health endpoint)
- **Health check interval:** 30 seconds
- **Health check timeout:** 5 seconds
- **Healthy threshold:** 2 consecutive successes
- **Unhealthy threshold:** 2 consecutive failures

**Why HTTPS (Port 443)?**

- Secure communication between users and ALB
- SSL/TLS certificate required (can be managed via AWS Certificate Manager)
- ALB terminates SSL, communicates with containers via HTTP on port 3000

---

### Architecture Overview

```
                    Internet
                       ↓
                       ↓ HTTPS (443)
                       ↓
┌──────────────────────────────────────┐
│   Application Load Balancer (ALB)   │
│     Security Group: schoolApp-alb-sg │
│     Public Subnets                   │
└──────────────────┬───────────────────┘
                   │
                   │ HTTP (3000)
                   │
        ┌──────────┴──────────┐
        │                     │
        ↓                     ↓
┌───────────────┐     ┌───────────────┐
│  ECS Task 1   │     │  ECS Task 2   │
│  Container    │     │  Container    │
│  Port 3000    │     │  Port 3000    │
│  Private      │     │  Private      │
│  Subnet 1     │     │  Subnet 2     │
└───────┬───────┘     └───────┬───────┘
        │                     │
        │ Security Group: schoolApp-ecs-sg
        │
        └─────────┬───────────┘
                  │
                  │ MySQL (3306)
                  ↓
        ┌─────────────────┐
        │   RDS Database  │
        │   Private       │
        │   Subnet        │
        └─────────────────┘
```

---

### Step 4: Service Configuration Summary

**Created Service with:**

- **Name:** `schoolapp-backend-service`
- **Cluster:** `schoolapp-cluster`
- **Launch Type:** AWS Fargate
- **Task Definition:** `schoolapp-backend-task:latest`
- **Desired Count:** 2 tasks
- **Deployment Type:** Rolling update
- **VPC:** Your VPC
- **Subnets:** 2 private subnets (different availability zones)
- **Security Group:** `schoolApp-ecs-sg`
- **Load Balancer:** Application Load Balancer on port 443
- **Target Port:** 3000 (container port)

---

### Deployment Process

When the service is created, ECS performs these steps:

```
1. Service created → ECS scheduler activated

2. Scheduler places 2 tasks across private subnets
   - Task 1 → Private Subnet A (AZ 1)
   - Task 2 → Private Subnet B (AZ 2)

3. ECS agent pulls image from ECR
   └── Uses Task Execution Role

4. Containers start with environment variables

5. Application starts (node index.js)
   └── Connects to RDS, Secrets Manager, Parameter Store

6. Health checks begin
   └── ALB sends requests to /health endpoint

7. Tasks pass health checks → Registered with target group

8. ALB starts routing traffic to healthy tasks

9. Service running → 2 tasks behind load balancer
```

---

### High Availability Benefits

**With 2 Tasks Across 2 Availability Zones:**

1. **Fault Tolerance:**
   - If one AZ fails, other task continues serving traffic
   - If one task fails, ECS automatically replaces it

2. **Load Distribution:**
   - ALB distributes requests across both tasks
   - Reduces load on individual containers

3. **Zero Downtime Deployments:**
   - Rolling update starts new tasks before stopping old ones
   - Always at least 2 healthy tasks during deployment

4. **Automatic Recovery:**
   - Failed tasks are automatically restarted
   - Unhealthy tasks are removed from load balancer rotation

---

### Verification

**Check Service Status:**

```bash
aws ecs describe-services \
  --cluster schoolapp-cluster \
  --services schoolapp-backend-service \
  --region us-east-1
```

**View Running Tasks:**

```bash
aws ecs list-tasks \
  --cluster schoolapp-cluster \
  --service-name schoolapp-backend-service \
  --region us-east-1
```

**Check Task Health:**

```bash
aws ecs describe-tasks \
  --cluster schoolapp-cluster \
  --tasks <task-arn> \
  --region us-east-1
```

**View Service Events:**

```bash
aws ecs describe-services \
  --cluster schoolapp-cluster \
  --services schoolapp-backend-service \
  --region us-east-1 \
  --query 'services[0].events[0:5]'
```

---

### Service Management

**Update Service (Change Task Count):**

```bash
aws ecs update-service \
  --cluster schoolapp-cluster \
  --service schoolapp-backend-service \
  --desired-count 3 \
  --region us-east-1
```

**Force New Deployment (Pull Latest Image):**

```bash
aws ecs update-service \
  --cluster schoolapp-cluster \
  --service schoolapp-backend-service \
  --force-new-deployment \
  --region us-east-1
```

**Stop Service (Scale to 0):**

```bash
aws ecs update-service \
  --cluster schoolapp-cluster \
  --service schoolapp-backend-service \
  --desired-count 0 \
  --region us-east-1
```

---

### Monitoring with CloudWatch

**View Container Logs:**

```bash
aws logs tail /ecs/schoolapp-backend --follow --region us-east-1
```

**View Specific Task Logs:**

```bash
aws logs tail /ecs/schoolapp-backend --follow \
  --filter-pattern "ERROR" \
  --region us-east-1
```

**CloudWatch Metrics to Monitor:**

- CPU utilization (should be < 80%)
- Memory utilization (should be < 80%)
- Target response time (ALB metrics)
- Healthy target count (should be 2)
- Unhealthy target count (should be 0)

---

### Common Service States

**ACTIVE:**

- Service is running
- Desired count matches running count
- All tasks are healthy

**DRAINING:**

- Service is being deleted or updated
- Tasks are being stopped gracefully

**INACTIVE:**

- Service has been deleted
- No tasks are running

---

### Troubleshooting Service Issues

**Issue 1: Tasks Keep Stopping and Restarting**

**Possible Causes:**

- Application crashes (check logs)
- Health check failing
- Insufficient memory/CPU
- Cannot connect to database

**Check Logs:**

```bash
aws logs tail /ecs/schoolapp-backend --follow --region us-east-1
```

**Issue 2: Tasks Not Registering with Load Balancer**

**Possible Causes:**

- Security group blocking traffic
- Health check endpoint not responding
- Container port mismatch

**Verify Security Groups:**

```bash
aws ec2 describe-security-groups \
  --group-names schoolApp-ecs-sg \
  --region us-east-1
```

**Issue 3: Service Won't Start Tasks**

**Possible Causes:**

- No available IP addresses in subnets
- Task execution role lacks permissions
- ECR image pull failed

**Check Service Events:**

```bash
aws ecs describe-services \
  --cluster schoolapp-cluster \
  --services schoolapp-backend-service \
  --region us-east-1 \
  --query 'services[0].events'
```

---

### Load Balancer Configuration Notes

**HTTPS Setup Requirements:**

1. **SSL/TLS Certificate:**
   - Must be issued or imported in AWS Certificate Manager
   - Certificate must match the domain name
   - Attached to ALB listener on port 443

2. **DNS Configuration:**
   - Create DNS record pointing to ALB DNS name
   - Example: `api.schoolapp.com` → `schoolapp-alb-123456.us-east-1.elb.amazonaws.com`

3. **Security Group Rules:**
   - ALB security group: Allow 443 from 0.0.0.0/0
   - ECS security group: Allow 3000 from ALB security group

**HTTP to HTTPS Redirect (Optional):**

- Create listener on port 80
- Redirect all HTTP traffic to HTTPS (port 443)
- Ensures all traffic is encrypted

---

### What Happens When User Makes a Request

```
1. User visits: https://api.schoolapp.com/api/register
   └── DNS resolves to ALB

2. Request hits ALB on port 443 (HTTPS)
   └── ALB terminates SSL/TLS

3. ALB forwards to target group (HTTP, port 3000)
   └── Chooses healthy task based on routing algorithm

4. Request reaches ECS task container on port 3000
   └── Your Node.js app receives the request

5. Application processes request
   ├── Validates input (userController)
   ├── Queries database (userModel + aws.js)
   └── Returns response

6. Response flows back through ALB to user
   └── ALB re-encrypts with SSL/TLS

7. User receives response (201 Created)
```

---

### Service is Now Running!

Your backend application is now:

- ✅ Running 2 containers across 2 availability zones
- ✅ Accessible via Application Load Balancer on HTTPS (port 443)
- ✅ Automatically recovers from failures
- ✅ Connected to RDS database
- ✅ Retrieving credentials from Parameter Store and Secrets Manager
- ✅ Logging to CloudWatch
- ✅ Ready to handle user registration requests

---

## Step 15: Configuring Application Load Balancer and HTTPS Access

This step covers creating the Application Load Balancer, configuring it to work with the ECS service, setting up HTTPS with SSL certificate, and configuring DNS routing with Route 53.

---

### Step 1: Create Application Load Balancer

Created an Application Load Balancer to route traffic to the ECS tasks.

**Via AWS Console:**

**1. Navigate to EC2 → Load Balancers**

- Click "Create Load Balancer"
- Select "Application Load Balancer"

**2. Basic Configuration:**

- **Name:** `schoolapp-alb`
- **Scheme:** Internet-facing
- **IP address type:** IPv4

**3. Network Mapping:**

- **VPC:** Selected your VPC
- **Availability Zones:** Selected 2 or more AZs with public subnets
  - Important: ALB must be in public subnets to accept internet traffic

**4. Security Group:**

- Selected `schoolApp-alb-sg`
- Allows inbound traffic on ports 80, 443, and 3000

**5. Listeners and Routing:**

- **Initial Listener:** HTTP on port 3000
- **Target Group:** Create new target group
  - Target type: IP
  - Protocol: HTTP
  - Port: 3000
  - Health check path: `/health`

**6. Create Load Balancer**

**ALB DNS Name:**

```
schoolapp-alb-92232072.us-east-1.elb.amazonaws.com
```

---

### Step 2: Assign ALB to ECS Service

After creating the ALB, assigned it to the ECS task definition/service.

**Via AWS Console:**

**Option A: During Service Creation**

- In the Load Balancer section
- Select "Application Load Balancer"
- Choose existing ALB: `schoolapp-alb`
- Select listener: Port 3000
- Select target group: The one created with ALB

**Option B: Update Existing Service**

- Navigate to ECS → Clusters → `schoolapp-cluster`
- Select service → Update
- Load balancing section
- Add load balancer: `schoolapp-alb`
- Configure listener and target group

---

### Step 3: Test Application on Port 3000

After the service started and tasks became healthy, tested the application.

**Test Command:**

```bash
curl http://schoolapp-alb-92232072.us-east-1.elb.amazonaws.com:3000/health
```

**Response:**

```json
{ "status": "OK" }
```

**✅ Success!** The application is running and accessible through the load balancer on port 3000.

**What This Confirms:**

- ECS tasks are running successfully
- Containers are listening on port 3000
- Security groups are configured correctly
- ALB can reach the containers
- Health checks are passing
- Target group has healthy targets

---

### Step 4: Configure HTTPS with SSL/TLS Certificate

To enable secure HTTPS access, added a new listener on port 443 with an SSL/TLS certificate.

**Prerequisites:**

- SSL/TLS certificate in AWS Certificate Manager (ACM)
- Certificate must be validated (DNS or email validation)
- Certificate must cover your domain name

**Add HTTPS Listener:**

**1. Navigate to Load Balancer:**

- EC2 → Load Balancers → `schoolapp-alb`
- Click "Listeners" tab

**2. Add Listener:**

- Click "Add listener"
- **Protocol:** HTTPS
- **Port:** 443
- **Default actions:** Forward to target group (same target group on port 3000)

**3. Secure Listener Settings:**

- **Security policy:** Recommended (ELBSecurityPolicy-TLS13-1-2-2021-06)
- **Default SSL/TLS certificate:**
  - From ACM (recommended)
  - Select your certificate from dropdown
  - Example: `*.schoolapp.com` or `api.schoolapp.com`

**4. Create Listener**

---

### Step 5: Redirect HTTP to HTTPS

To ensure all traffic uses HTTPS, configured the port 3000 listener to redirect to HTTPS 443.

**Modify Port 3000 Listener:**

**1. Navigate to Listeners:**

- EC2 → Load Balancers → `schoolapp-alb` → Listeners

**2. Edit Port 3000 Listener:**

- Select the listener on port 3000
- Click "Edit"

**3. Change Default Action:**

- Remove "Forward to" action
- Add "Redirect" action
  - **Protocol:** HTTPS
  - **Port:** 443
  - **Status code:** 301 (Permanent redirect)

**4. Save Changes**

**What This Does:**

```
User requests: http://schoolapp-alb-92232072.us-east-1.elb.amazonaws.com:3000/health
                ↓
ALB redirects: https://schoolapp-alb-92232072.us-east-1.elb.amazonaws.com/health
```

---

### Listener Configuration Summary

After configuration, the ALB has these listeners:

| Port | Protocol | Action                                            |
| ---- | -------- | ------------------------------------------------- |
| 3000 | HTTP     | Redirect to HTTPS 443 (301)                       |
| 443  | HTTPS    | Forward to target group (containers on port 3000) |

**Traffic Flow:**

```
Internet → ALB (HTTPS 443) → Target Group → ECS Tasks (HTTP 3000)
```

**Security:**

- External traffic: Encrypted (HTTPS)
- Internal traffic (ALB to containers): HTTP (within VPC, secure)
- ALB terminates SSL/TLS

---

### Step 6: Create DNS Record in Route 53

To access the application via a custom domain name instead of the ALB DNS name, created an A record in Route 53.

**Prerequisites:**

- Domain registered (in Route 53 or external registrar)
- Hosted zone created in Route 53

**Create A Record:**

**1. Navigate to Route 53:**

- Route 53 → Hosted zones
- Select your hosted zone (e.g., `schoolapp.com`)

**2. Create Record:**

- Click "Create record"

**3. Record Configuration:**

- **Record name:** `api` (or leave blank for root domain)
  - Results in: `api.schoolapp.com` or `schoolapp.com`
- **Record type:** A - IPv4 address
- **Alias:** ✅ Enabled
  - **Route traffic to:** Alias to Application and Classic Load Balancer
  - **Region:** us-east-1
  - **Load balancer:** `schoolapp-alb-92232072.us-east-1.elb.amazonaws.com`
- **Routing policy:** Simple routing
- **Evaluate target health:** ✅ Enabled (recommended)

**4. Create Record**

**Why Use Alias Record?**

- ✅ No charge for alias queries (standard A records may have charges)
- ✅ AWS automatically updates IP addresses if ALB IPs change
- ✅ Supports health checks
- ✅ Works with zone apex (root domain)

---

### DNS Propagation

After creating the A record, DNS changes take time to propagate.

**Propagation Timeline:**

- Route 53 name servers: Immediate to a few minutes
- Global DNS: Up to 48 hours (typically 5-30 minutes)

**Check DNS Propagation:**

```bash
# Check if DNS is resolving
nslookup api.schoolapp.com

# Test with dig
dig api.schoolapp.com

# Test HTTPS access
curl https://api.schoolapp.com/health
```

**Expected Response:**

```json
{ "status": "OK" }
```

---

### Complete Access URLs

Your application is now accessible via:

**1. ALB DNS Name (Port 3000 - Redirects to HTTPS):**

```
http://schoolapp-alb-92232072.us-east-1.elb.amazonaws.com:3000/health
  ↓ Redirects to ↓
https://schoolapp-alb-92232072.us-east-1.elb.amazonaws.com/health
```

**2. ALB DNS Name (HTTPS):**

```
https://schoolapp-alb-92232072.us-east-1.elb.amazonaws.com/health
```

**3. Custom Domain (HTTPS):**

```
https://api.schoolapp.com/health
```

---

### Verification Commands

**Test Health Endpoint:**

```bash
# Via ALB DNS
curl https://schoolapp-alb-92232072.us-east-1.elb.amazonaws.com/health

# Via custom domain
curl https://api.schoolapp.com/health
```

**Test Registration Endpoint:**

```bash
curl -X POST https://api.schoolapp.com/api/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "phone": "1234567890",
    "program": "Computer Science"
  }'
```

**Check SSL Certificate:**

```bash
# View certificate details
openssl s_client -connect api.schoolapp.com:443 -servername api.schoolapp.com
```

---

### Architecture Overview

```
                    Internet Users
                          ↓
         ┌────────────────────────────┐
         │  Route 53 DNS Resolution   │
         │  api.schoolapp.com →       │
         │  schoolapp-alb.elb.aws...  │
         └────────────────┬───────────┘
                          ↓
                    HTTPS (443)
                          ↓
         ┌────────────────────────────┐
         │  Application Load Balancer │
         │  - SSL/TLS Termination     │
         │  - Port 3000 → Redirect    │
         │  - Port 443 → Forward      │
         └────────────────┬───────────┘
                          ↓
                    HTTP (3000)
                          ↓
         ┌────────────────┴───────────┐
         │                            │
    ┌────▼────┐                 ┌────▼────┐
    │ ECS     │                 │ ECS     │
    │ Task 1  │                 │ Task 2  │
    │ Port    │                 │ Port    │
    │ 3000    │                 │ 3000    │
    └────┬────┘                 └────┬────┘
         │                            │
         └────────────┬───────────────┘
                      ↓
              MySQL (3306)
                      ↓
         ┌────────────────────┐
         │   RDS Database     │
         └────────────────────┘
```

---

### Security Best Practices Implemented

**1. HTTPS Everywhere:**

- ✅ All external traffic encrypted with SSL/TLS
- ✅ Certificate from AWS Certificate Manager
- ✅ HTTP automatically redirects to HTTPS

**2. Security Groups:**

- ✅ ALB security group allows 443 from internet
- ✅ ECS security group allows 3000 only from ALB
- ✅ RDS security group allows 3306 only from ECS

**3. Private Resources:**

- ✅ ECS tasks in private subnets
- ✅ RDS in private subnet
- ✅ Only ALB is publicly accessible

**4. DNS Alias:**

- ✅ Custom domain with proper SSL certificate
- ✅ Health checks enabled
- ✅ Automatic IP address updates

---

### Monitoring and Management

**Check ALB Health:**

```bash
aws elbv2 describe-load-balancers \
  --names schoolapp-alb \
  --region us-east-1
```

**Check Target Health:**

```bash
aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn> \
  --region us-east-1
```

**View ALB Access Logs (if enabled):**

- S3 bucket with ALB access logs
- Shows all requests, latencies, status codes

**CloudWatch Metrics:**

- Target response time
- Request count
- Healthy/Unhealthy target count
- HTTP 4xx/5xx errors

---

### Application is Now Production-Ready!

Your backend application is now:

- ✅ Running on AWS ECS with Fargate
- ✅ Behind Application Load Balancer for high availability
- ✅ Accessible via HTTPS with valid SSL certificate
- ✅ Using custom domain name (api.schoolapp.com)
- ✅ Automatically redirects HTTP to HTTPS
- ✅ Connected to RDS database securely
- ✅ Retrieving credentials from Parameter Store and Secrets Manager
- ✅ Health checks passing
- ✅ Ready to handle production traffic
