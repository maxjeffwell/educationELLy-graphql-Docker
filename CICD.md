# CI/CD Pipeline Documentation

This document explains the CI/CD setup for the educationELLy GraphQL project, including GitHub Actions workflows, Docker Hub integration, and deployment strategies.

## Overview

The CI/CD pipeline automatically:
1. Tests and lints code on every push/PR
2. Builds Docker images for both server and client
3. Pushes images to Docker Hub
4. Scans for security vulnerabilities
5. Tags images appropriately for versioning

## Architecture

```
GitHub Push/PR
     ↓
CI Workflow (test, lint, build check)
     ↓
✅ Passes?
     ├─ NO: Fail, block merge
     └─ YES (on master/main only)
         ↓
Docker Build & Push
  - Build multi-platform images (amd64, arm64)
  - Push to Docker Hub
  - Run Trivy security scan
     ↓
Images Available on Docker Hub
     ↓
Deploy (manual or automated)
```

## GitHub Actions Workflows

### Server Repository Workflows

#### 1. CI Workflow (`.github/workflows/ci.yml`)

**Triggers:**
- Push to `master`, `main`, or `develop`
- Pull requests to these branches

**Jobs:**
- **test-and-lint**: Runs on Node.js 18.x and 20.x
  - Install dependencies
  - Run ESLint
  - Run tests (if available)
  - Test environment variables

- **docker-build-check**: Validates Docker build without pushing
  - Builds Docker image
  - Tests container startup
  - Validates health checks

- **ci-success**: Summary job that ensures all checks passed

#### 2. Docker Build & Push (`.github/workflows/docker-build-push.yml`)

**Triggers:**
- Push to `master` or `main`
- Semantic version tags (`v*.*.*`)
- Manual workflow dispatch

**Features:**
- Multi-platform builds (linux/amd64, linux/arm64)
- Smart tagging:
  - `latest` - Latest commit on default branch
  - `master` - Current branch name
  - `v1.2.3` - Semantic versions
  - `master-abc1234` - Branch with commit SHA
- GitHub Actions cache for faster rebuilds
- Trivy security vulnerability scanning
- SARIF upload to GitHub Security tab

**Docker Image:** `maxjeffwell/educationelly-graphql-server`

### Client Repository Workflows

#### 1. CI Workflow (`.github/workflows/ci.yml`)

**Triggers:**
- Push to `master`, `main`, or `develop`
- Pull requests to these branches

**Jobs:**
- Runs on Node.js 18.x and 20.x
- Linting with ESLint
- Format checking with Prettier
- Type checking (placeholder)
- Tests with coverage
- Production build verification
- Codecov upload

#### 2. Docker Build & Push (`.github/workflows/docker-build-push.yml`)

**Triggers:**
- Push to `master` or `main`
- Semantic version tags (`v*.*.*`)
- Manual workflow dispatch

**Features:**
- Same as server workflow
- Includes `REACT_APP_API_BASE_URL` build argument
- Multi-stage build with nginx

**Docker Image:** `maxjeffwell/educationelly-graphql-client`

## Docker Hub Setup

### 1. Create Docker Hub Account

If you don't have one already:
1. Go to https://hub.docker.com/signup
2. Create an account
3. Verify your email

### 2. Create Docker Hub Repositories

Create two public repositories:
- `maxjeffwell/educationelly-graphql-server`
- `maxjeffwell/educationelly-graphql-client`

Steps:
1. Log in to Docker Hub
2. Click "Create Repository"
3. Enter repository name
4. Set visibility to "Public" (or "Private" if preferred)
5. Add description
6. Click "Create"

### 3. Generate Access Token

GitHub Actions needs an access token to push images:

1. Log in to Docker Hub
2. Go to **Account Settings** → **Security** → **Access Tokens**
3. Click "New Access Token"
4. Name: `github-actions-educationelly`
5. Permissions: **Read & Write**
6. Click "Generate"
7. **Copy the token immediately** (you won't see it again!)

## GitHub Secrets Configuration

### Server Repository Secrets

Go to `https://github.com/maxjeffwell/educationELLy-graphql-server/settings/secrets/actions`

Add these secrets:

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `DOCKERHUB_USERNAME` | `maxjeffwell` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | `dckr_pat_...` | Access token from Docker Hub |
| `MONGODB_URI_TEST` | `mongodb://localhost:27017/test` | Optional: Test database URI |

### Client Repository Secrets

Go to `https://github.com/maxjeffwell/educationELLy-graphql-client/settings/secrets/actions`

Add these secrets:

| Secret Name | Value | Description |
|-------------|-------|-------------|
| `DOCKERHUB_USERNAME` | `maxjeffwell` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | `dckr_pat_...` | Access token from Docker Hub |
| `CODECOV_TOKEN` | `...` | Optional: Codecov upload token |

### Adding Secrets

1. Navigate to repository settings
2. Click **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Enter name and value
5. Click **Add secret**

## Docker Image Tagging Strategy

Images are automatically tagged based on the trigger:

### Branch Pushes
```
maxjeffwell/educationelly-graphql-server:master
maxjeffwell/educationelly-graphql-server:latest
maxjeffwell/educationelly-graphql-server:master-abc1234
```

### Semantic Version Tags
```bash
git tag v1.2.3
git push origin v1.2.3
```
Creates:
```
maxjeffwell/educationelly-graphql-server:1.2.3
maxjeffwell/educationelly-graphql-server:1.2
maxjeffwell/educationelly-graphql-server:1
maxjeffwell/educationelly-graphql-server:latest
```

## Pulling Images from Docker Hub

Once CI/CD is set up, anyone can pull your images:

```bash
# Latest version
docker pull maxjeffwell/educationelly-graphql-server:latest
docker pull maxjeffwell/educationelly-graphql-client:latest

# Specific version
docker pull maxjeffwell/educationelly-graphql-server:1.2.3
docker pull maxjeffwell/educationelly-graphql-client:1.2.3

# Specific commit
docker pull maxjeffwell/educationelly-graphql-server:master-abc1234
```

## Using Docker Hub Images in docker-compose

Update your `docker-compose.yml` to use Docker Hub images instead of building locally:

```yaml
version: '3.8'

services:
  server:
    image: maxjeffwell/educationelly-graphql-server:latest
    # Remove 'build' section
    environment:
      - MONGODB_URI=${MONGODB_URI}
      - JWT_SECRET=${JWT_SECRET}
    ports:
      - "8000:8000"

  client:
    image: maxjeffwell/educationelly-graphql-client:latest
    # Remove 'build' section
    ports:
      - "80:80"
    depends_on:
      - server
```

Then simply:
```bash
docker-compose pull  # Pull latest images
docker-compose up -d # Start containers
```

## Security Scanning

### Trivy Vulnerability Scanner

Every Docker build automatically scans for:
- Critical vulnerabilities
- High-severity vulnerabilities
- Known CVEs in dependencies and base images

Results are:
- Uploaded to GitHub Security tab
- Available in workflow logs
- Can be reviewed before deployment

### Viewing Security Results

1. Go to repository on GitHub
2. Click **Security** tab
3. Click **Code scanning**
4. View Trivy scan results

### Failing on Vulnerabilities

To fail the build on critical vulnerabilities, modify the workflow:

```yaml
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: ${{ env.DOCKER_IMAGE }}:${{ github.ref_name }}
    format: 'sarif'
    output: 'trivy-results.sarif'
    severity: 'CRITICAL,HIGH'
    exit-code: '1'  # Add this line
```

## Manual Workflow Triggers

You can manually trigger Docker builds:

1. Go to repository on GitHub
2. Click **Actions** tab
3. Select "Build and Push Docker Image" workflow
4. Click **Run workflow**
5. Select branch
6. Click **Run workflow**

## Monitoring CI/CD

### Workflow Status

Check workflow status:
- Repository homepage shows status badges
- Actions tab shows all workflow runs
- Email notifications for failures (configure in Settings)

### Docker Hub

Monitor your images:
- https://hub.docker.com/r/maxjeffwell/educationelly-graphql-server
- https://hub.docker.com/r/maxjeffwell/educationelly-graphql-client

View:
- Download statistics
- Image sizes
- Available tags
- Vulnerability scans (Docker Hub Pro feature)

## Troubleshooting

### Build Fails: Authentication

**Error:** `unauthorized: authentication required`

**Solution:**
- Check `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` secrets
- Verify token has Read & Write permissions
- Regenerate token if expired

### Build Fails: Platform Issues

**Error:** `multiple platforms feature is currently not supported`

**Solution:**
- Remove `platforms` line from workflow
- Or use only: `platforms: linux/amd64`

### Trivy Scan Fails

**Error:** `failed to scan image`

**Solution:**
- Check if image was pushed successfully
- Verify image tag in Trivy step matches built image
- May need to pull image before scanning

### Rate Limiting

Docker Hub has pull rate limits:
- Anonymous: 100 pulls per 6 hours
- Authenticated: 200 pulls per 6 hours
- Pro: Unlimited

**Solution:** Always authenticate in workflows (already configured)

## Best Practices

### 1. Semantic Versioning

Use semantic versioning for releases:
```bash
git tag v1.2.3
git push origin v1.2.3
```

### 2. Protected Branches

Protect `master`/`main` branches:
- Require PR reviews
- Require status checks (CI must pass)
- No direct pushes

### 3. Secrets Rotation

Rotate secrets regularly:
- Generate new Docker Hub token every 90 days
- Update GitHub secrets
- Revoke old token

### 4. Image Cleanup

Clean up old images periodically:
- Delete unused tags from Docker Hub
- Keep last 10-20 versions
- Always keep `latest` and semantic versions

### 5. Multi-Stage Builds

Use multi-stage Dockerfiles (already implemented):
- Separate `development` and `production` stages
- Smaller production images
- Faster builds with caching

## Next Steps

1. **Set up GitHub Secrets** (both repos)
2. **Test workflows** by pushing a commit
3. **Verify images** appear on Docker Hub
4. **Create a release** with semantic version tag
5. **Update docker-compose** to use Hub images
6. **Deploy to production** using pulled images

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Hub Documentation](https://docs.docker.com/docker-hub/)
- [Semantic Versioning](https://semver.org/)
- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
