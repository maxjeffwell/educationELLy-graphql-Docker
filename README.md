# educationELLy GraphQL

[![Server CI](https://github.com/maxjeffwell/educationELLy-graphql-server/actions/workflows/ci.yml/badge.svg)](https://github.com/maxjeffwell/educationELLy-graphql-server/actions/workflows/ci.yml)
[![Server Docker](https://github.com/maxjeffwell/educationELLy-graphql-server/actions/workflows/docker-build-push.yml/badge.svg)](https://github.com/maxjeffwell/educationELLy-graphql-server/actions/workflows/docker-build-push.yml)
[![Client CI](https://github.com/maxjeffwell/educationELLy-graphql-client/actions/workflows/ci.yml/badge.svg)](https://github.com/maxjeffwell/educationELLy-graphql-client/actions/workflows/ci.yml)
[![Client Docker](https://github.com/maxjeffwell/educationELLy-graphql-client/actions/workflows/docker-build-push.yml/badge.svg)](https://github.com/maxjeffwell/educationELLy-graphql-client/actions/workflows/docker-build-push.yml)

A full-stack application for managing English Language Learner (ELL) student educational data, built with React and Apollo GraphQL.

## Architecture

This is a monorepo containing two separate applications:

- **[educationELLy-graphql-server](./educationELLy-graphql-server/)**: Apollo GraphQL API server (Node.js + Express + MongoDB)
- **[educationELLy-graphql-client](./educationELLy-graphql-client/)**: React single-page application (React 18 + Apollo Client)

## Tech Stack

### Backend
- Apollo Server 4.x
- Express.js
- MongoDB with Mongoose
- JWT Authentication
- GraphQL

### Frontend
- React 18.3.1
- Apollo Client 3.11.8
- React Router 6.28.0
- Semantic UI React
- styled-components

## Quick Start

### Local Development (Without Docker)

1. **Setup Server**
   ```bash
   cd educationELLy-graphql-server
   npm install
   cp .env.example .env  # Configure your MongoDB URI and JWT secret
   npm run dev
   ```

2. **Setup Client**
   ```bash
   cd educationELLy-graphql-client
   npm install
   cp .env.example .env  # Configure API endpoint
   npm start
   ```

3. **Access Application**
   - Frontend: http://localhost:3000
   - GraphQL API: http://localhost:8000/graphql

### Docker Deployment

Run the entire stack with Docker:

```bash
# Development
docker-compose up -d

# Production
cp .env.production.example .env.production  # Configure production values
docker-compose -f docker-compose.prod.yml --env-file .env.production up -d
```

See [DOCKER.md](./DOCKER.md) for detailed Docker deployment instructions.

## Deployment

### Current Deployments

- **Production Server**: https://educationelly-server-graphql-5b9748151d5a.herokuapp.com
- **Production Client**: https://educationelly-client-graphql-176ac5044d94.herokuapp.com

Both applications are currently deployed on Heroku.

### Deployment Options

1. **Heroku** (Current): Uses buildpacks, separate deployments for client and server
2. **Docker** (New): Containerized deployment with docker-compose
3. **Kubernetes** (Planned): Future deployment to Linode Kubernetes Engine

## Project Structure

```
educationELLy-graphql/
├── educationELLy-graphql-server/    # Backend API
│   ├── src/
│   │   ├── models/                  # Mongoose models
│   │   ├── schema/                  # GraphQL schemas
│   │   ├── resolvers/               # GraphQL resolvers
│   │   ├── loaders/                 # DataLoader instances
│   │   └── index.js                 # Server entry point
│   ├── Dockerfile
│   └── package.json
│
├── educationELLy-graphql-client/    # Frontend SPA
│   ├── src/
│   │   ├── components/              # React components
│   │   └── index.js                 # App entry point
│   ├── Dockerfile
│   └── package.json
│
├── nginx/                           # Nginx reverse proxy config
│   └── nginx.conf
│
├── k8s/                             # Kubernetes manifests
│   ├── namespace.yaml
│   ├── configmap.yaml
│   ├── secrets.yaml.example
│   ├── server-deployment.yaml
│   ├── server-service.yaml
│   ├── client-deployment.yaml
│   ├── client-service.yaml
│   ├── ingress.yaml
│   └── hpa.yaml
│
├── docker-compose.yml               # Development Docker setup
├── docker-compose.prod.yml          # Production Docker setup
├── .env                             # Development environment variables
├── .env.production.example          # Production environment template
├── DOCKER.md                        # Docker deployment guide
├── KUBERNETES.md                    # Kubernetes deployment guide
└── README.md                        # This file
```

## Features

### Authentication & Authorization
- JWT-based authentication
- Role-based access control (Admin, Teacher, Student)
- Secure password hashing with bcrypt

### Student Management
- Create, read, update, delete student records
- Track student progress and assessments
- Manage student profiles and educational data

### API Features
- GraphQL API with Apollo Server
- DataLoader for efficient batch loading
- Health check endpoints
- CORS configuration
- Error handling and logging

### Frontend Features
- Responsive UI with Semantic UI React
- Client-side routing with React Router
- Session management with Apollo Client
- Protected routes
- Error boundaries

## Development

### Server Development

```bash
cd educationELLy-graphql-server
npm run dev          # Start dev server with hot reload
npm run lint         # Run ESLint
npm run lint:fix     # Fix linting issues
npm test             # Run tests
```

### Client Development

```bash
cd educationELLy-graphql-client
npm start            # Start dev server (port 3000)
npm run build        # Create production build
npm run lint         # Run ESLint
npm run format       # Format with Prettier
npm run quality      # Run all quality checks
npm test             # Run tests
```

## Environment Variables

### Server
- `MONGODB_URI`: MongoDB connection string
- `JWT_SECRET`: Secret for JWT signing
- `NODE_ENV`: Environment (development/production)
- `PORT`: Server port (default: 8000)
- `ALLOWED_ORIGINS`: CORS allowed origins

### Client
- `REACT_APP_API_BASE_URL`: GraphQL API endpoint

## CI/CD

The project includes automated CI/CD pipelines with GitHub Actions:

- **Continuous Integration**: Automated testing, linting, and build verification
- **Docker Hub**: Automatic image builds and publishing
- **Security Scanning**: Trivy vulnerability scanning
- **Multi-Platform**: Images built for linux/amd64 and linux/arm64

### Docker Hub Images

Pre-built images are available:
- Server: `maxjeffwell/educationelly-graphql-server`
- Client: `maxjeffwell/educationelly-graphql-client`

```bash
docker pull maxjeffwell/educationelly-graphql-server:latest
docker pull maxjeffwell/educationelly-graphql-client:latest
```

## Documentation

- [Docker Deployment Guide](./DOCKER.md) - Run with Docker Compose
- [Kubernetes Deployment Guide](./KUBERNETES.md) - Deploy to K8s (Linode LKE)
- [CI/CD Pipeline Guide](./CICD.md) - GitHub Actions setup
- [Server Documentation](./educationELLy-graphql-server/README.md)
- [Client Documentation](./educationELLy-graphql-client/README.md)
- [Client Claude Guide](./educationELLy-graphql-client/CLAUDE.md)

## CI/CD

The client includes a GitHub Actions workflow that runs on push/PR:
- Linting (ESLint)
- Code formatting checks (Prettier)
- Tests with coverage
- Production build verification
- Coverage upload to Codecov

## Security Considerations

- Never commit `.env` or `.env.production` files
- Use strong JWT secrets (64+ characters)
- Configure CORS to only allow trusted origins
- Keep MongoDB credentials secure
- Rotate secrets periodically
- Use different credentials for dev and production

## Contributing

1. Each directory (client/server) has its own linting and formatting rules
2. Use the provided npm scripts for quality checks
3. Follow the existing code structure and patterns
4. Ensure all tests pass before submitting

## License

[Add your license information here]

## Support

[Add support/contact information here]
