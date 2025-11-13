# Docker Deployment Guide

This guide explains how to run the educationELLy full-stack application using Docker containers.

## Architecture Overview

The application consists of two main services:

- **Server** (`educationELLy-graphql-server`): Apollo GraphQL API running on Node.js
- **Client** (`educationELLy-graphql-client`): React SPA served by nginx with reverse proxy to the server

The client's nginx configuration proxies `/graphql` requests to the backend server, allowing both services to be accessed through a single entry point (port 80).

## Prerequisites

- Docker Engine 20.10+
- Docker Compose 2.0+

## Quick Start (Development)

1. **Create environment file**

   The `.env` file is already configured for local development. Review and update if needed:
   ```bash
   cat .env
   ```

2. **Build and start the containers**

   ```bash
   docker-compose up -d
   ```

3. **Access the application**

   - Frontend: http://localhost
   - GraphQL API: http://localhost/graphql
   - Backend health check: http://localhost/api/health

4. **View logs**

   ```bash
   # All services
   docker-compose logs -f

   # Specific service
   docker-compose logs -f server
   docker-compose logs -f client
   ```

5. **Stop the containers**

   ```bash
   docker-compose down
   ```

## Production Deployment

For production deployments, use the production-specific configuration:

1. **Create production environment file**

   ```bash
   cp .env.production.example .env.production
   ```

   Edit `.env.production` with your production values:
   - **MONGODB_URI**: Your production MongoDB connection string
   - **JWT_SECRET**: Generate using `node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"`
   - **ALLOWED_ORIGINS**: Your production domain(s)
   - **NODE_ENV**: Set to `production`

2. **Build and start with production configuration**

   ```bash
   docker-compose -f docker-compose.prod.yml --env-file .env.production up -d
   ```

3. **Production features include:**
   - Resource limits (CPU and memory)
   - Restart policy set to `always`
   - Enhanced health checks with longer timeouts
   - Log rotation configured
   - Security hardening (read-only filesystems, no-new-privileges)
   - Service dependency management with health check conditions
   - Versioning support via `VERSION` environment variable

4. **Tagging and versioning**

   ```bash
   # Build with version tag
   VERSION=1.0.0 docker-compose -f docker-compose.prod.yml build

   # Deploy specific version
   VERSION=1.0.0 docker-compose -f docker-compose.prod.yml --env-file .env.production up -d
   ```

5. **Production monitoring**

   ```bash
   # Check status
   docker-compose -f docker-compose.prod.yml ps

   # View logs with timestamps
   docker-compose -f docker-compose.prod.yml logs -f --timestamps

   # Check resource usage
   docker stats educationelly-server-prod educationelly-client-prod
   ```

## Docker Commands

### Building

```bash
# Build all services
docker-compose build

# Build specific service
docker-compose build server
docker-compose build client

# Force rebuild without cache
docker-compose build --no-cache
```

### Running

```bash
# Start in detached mode
docker-compose up -d

# Start in foreground (see logs)
docker-compose up

# Start specific service
docker-compose up -d server
```

### Managing

```bash
# Stop containers (keep volumes)
docker-compose stop

# Start stopped containers
docker-compose start

# Restart containers
docker-compose restart

# Remove containers (keeps images and volumes)
docker-compose down

# Remove containers and volumes
docker-compose down -v

# Remove containers, volumes, and images
docker-compose down -v --rmi all
```

### Monitoring

```bash
# Check container status
docker-compose ps

# View logs
docker-compose logs -f

# Execute commands in running container
docker-compose exec server sh
docker-compose exec client sh

# View resource usage
docker stats
```

## Environment Variables

### Server Environment Variables

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| `MONGODB_URI` | MongoDB connection string | Yes | - |
| `JWT_SECRET` | Secret key for JWT signing | Yes | - |
| `NODE_ENV` | Environment mode | No | `production` |
| `PORT` | Server port | No | `8000` |
| `ALLOWED_ORIGINS` | CORS allowed origins | No | `http://localhost` |

### Client Build Arguments

| Variable | Description | Default |
|----------|-------------|---------|
| `REACT_APP_API_BASE_URL` | GraphQL API endpoint | `/graphql` |

## Heroku Deployment (Unchanged)

Your existing Heroku deployment continues to work as before:

- **Server**: https://educationelly-server-graphql-5b9748151d5a.herokuapp.com
- **Client**: https://educationelly-client-graphql-176ac5044d94.herokuapp.com

The Docker setup is independent and doesn't affect your Heroku deployment.

## Kubernetes Deployment (Linode)

For deploying to Kubernetes on Linode, you'll need to:

1. **Push images to a container registry**

   ```bash
   # Tag images
   docker tag educationelly-server:latest your-registry/educationelly-server:latest
   docker tag educationelly-client:latest your-registry/educationelly-client:latest

   # Push to registry
   docker push your-registry/educationelly-server:latest
   docker push your-registry/educationelly-client:latest
   ```

2. **Create Kubernetes manifests**

   You'll need:
   - Deployment manifests for server and client
   - Service manifests to expose the pods
   - ConfigMap for non-sensitive configuration
   - Secret for sensitive data (MONGODB_URI, JWT_SECRET)
   - Ingress for external access
   - PersistentVolumeClaim if needed

3. **Key considerations for K8s**
   - Use Secrets for sensitive environment variables
   - Set resource limits and requests
   - Configure liveness and readiness probes (health checks are already in place)
   - Consider using a LoadBalancer or Ingress controller
   - Set up horizontal pod autoscaling if needed

## Health Checks

Both services have health check endpoints:

- **Client**: `http://localhost/health` (returns 200 OK)
- **Server**: `http://localhost/api/health` (proxied from backend)

Health checks are configured in docker-compose.yml and run automatically.

## Networking

Services communicate over a bridge network named `educationelly-network`:

- Server is accessible internally as `server:8000`
- Client proxies GraphQL requests to `http://server:8000/graphql`
- Only the client's port 80 is exposed to the host

## Troubleshooting

### Container won't start

```bash
# Check logs
docker-compose logs server
docker-compose logs client

# Check if ports are in use
lsof -i :80
lsof -i :8000
```

### Database connection issues

- Verify MONGODB_URI is correct in .env
- Check if MongoDB Atlas allows connections from Docker host IP
- Ensure network_mode allows external connections

### Build failures

```bash
# Clean build
docker-compose down
docker-compose build --no-cache
docker-compose up -d
```

### Client can't reach server

- Verify both containers are on the same network: `docker network inspect educationelly-network`
- Check nginx configuration in `nginx/nginx.conf`
- Ensure server container name is `server` in upstream configuration

## Development vs Production

### Configuration Files

- **`docker-compose.yml`**: Development configuration with `.env` file
- **`docker-compose.prod.yml`**: Production configuration with `.env.production` file

### Key Differences

| Feature | Development | Production |
|---------|-------------|------------|
| **Environment File** | `.env` | `.env.production` |
| **Resource Limits** | None | CPU & Memory limits configured |
| **Restart Policy** | `unless-stopped` | `always` |
| **Container Names** | `educationelly-*` | `educationelly-*-prod` |
| **Health Checks** | Standard intervals | Enhanced with longer timeouts |
| **Logging** | Default | Log rotation enabled (10MB, 3 files) |
| **Security** | Basic | Hardened (read-only FS, no privileges) |
| **Image Versioning** | `latest` | Supports `VERSION` env var |

### Running Different Environments

```bash
# Development
docker-compose up -d

# Production
docker-compose -f docker-compose.prod.yml --env-file .env.production up -d
```

## Security Notes

- **Never commit `.env` or `.env.production` files** with real credentials to version control
- Add `.env.production` to your `.gitignore` file
- Use strong, randomly generated JWT_SECRET in production (64+ characters recommended)
- Configure ALLOWED_ORIGINS to only include trusted domains (no wildcards in production)
- Keep MongoDB connection string secure with strong passwords
- For Kubernetes/production, consider using Docker secrets or K8s secrets instead of environment files
- Ensure MongoDB Atlas IP whitelist includes your deployment server IPs
- Rotate JWT secrets periodically
- Use different database credentials for development and production
- Enable MongoDB Atlas network access restrictions
- Consider implementing rate limiting at the nginx level for production

## CI/CD Integration

This project includes automated CI/CD pipelines that:
- Automatically build Docker images on every push to `master`/`main`
- Push images to Docker Hub
- Run security scans with Trivy
- Support multi-platform builds (amd64, arm64)
- Tag images with semantic versions

### Quick Start with Docker Hub Images

Instead of building locally, you can pull pre-built images:

```bash
# Pull latest images from Docker Hub
docker pull maxjeffwell/educationelly-graphql-server:latest
docker pull maxjeffwell/educationelly-graphql-client:latest

# Run with docker-compose using Hub images
docker-compose pull
docker-compose up -d
```

### Available Images

- **Server**: `maxjeffwell/educationelly-graphql-server`
- **Client**: `maxjeffwell/educationelly-graphql-client`

### For Complete CI/CD Documentation

See [CICD.md](./CICD.md) for:
- GitHub Actions workflows setup
- Docker Hub configuration
- GitHub Secrets setup
- Security scanning details
- Deployment strategies

## Next Steps

- ✅ CI/CD pipeline configured
- ✅ Docker Hub integration complete
- Add Kubernetes manifests for Linode deployment
- Configure monitoring and logging (Prometheus, Grafana, ELK stack)
- Implement automated backups for MongoDB
- Add SSL/TLS termination at load balancer or ingress level
