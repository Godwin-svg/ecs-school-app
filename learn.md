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
