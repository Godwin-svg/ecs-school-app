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
