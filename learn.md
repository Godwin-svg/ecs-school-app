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
