# AGENTS.md

> AI Agent Documentation for educationELLy GraphQL Project

This document provides comprehensive information about the educationELLy GraphQL project, including its architecture, technology stack, coding standards, and development workflows. This documentation is designed to help AI agents understand and work with the project effectively.

---

## Project Overview

### Description
**educationELLy GraphQL** is a full-stack application for managing English Language Learner (ELL) student educational data. The application is built with modern web technologies including React and Apollo GraphQL, following a monorepo architecture with separate client and server applications.

### Business Domain
- **Domain**: Education Technology (EdTech)
- **Focus**: English Language Learner (ELL) student management
- **License**: GNU GPLv3
- **Author**: Jeff Maxwell (maxjeffwell@gmail.com)

### Target Users
- **Teachers**: Manage student data and track progress
- **Administrators**: Oversee system operations and user management
- **Students**: Access their educational profiles and progress

### Core Functionality
- Student profile management
- Educational data tracking
- Progress assessment and monitoring
- Role-based access control (Admin, Teacher, Student)

### Architecture
The project follows a **monorepo structure** with two main applications:

1. **Backend (educationELLy-graphql-server)**
   - Apollo GraphQL API
   - Node.js + Express + MongoDB stack
   - JWT authentication with role-based authorization
   - DataLoader for efficient batch loading
   - Health check endpoints
   - CORS configuration

2. **Frontend (educationELLy-graphql-client)**
   - React Single Page Application (SPA)
   - React 18 + Apollo Client + Semantic UI
   - Client-side routing with React Router 6
   - Session management with localStorage
   - Protected routes with authentication HOCs

---

## Technology Stack

### Languages & Runtime
- **JavaScript**: ES2022/ES2021 (modern ECMAScript)
- **Runtime**: Node.js 18+ (backend)
- **Module System**: ES Modules

### Backend Technologies
- **Framework**: Express.js 4.17.1
- **GraphQL Server**: Apollo Server 4.12.2
- **GraphQL**: GraphQL 16.11.0
- **Database**: MongoDB with Mongoose 8.16.3 ODM
- **Authentication**:
  - JWT (jsonwebtoken 9.0.2)
  - Password hashing with bcryptjs 2.4.3
- **Data Optimization**: DataLoader 1.4.0 (batch loading and caching)

### Frontend Technologies
- **Framework**: React 18.3.1
- **GraphQL Client**: Apollo Client 3.11.8
- **Routing**: React Router 6.28.0
- **UI Library**: Semantic UI React 2.1.5
- **Styling**: styled-components 5.3.11
- **Build Tool**: Create React App 5.0.1 (react-scripts)

### Testing Frameworks
- **Backend**: Mocha 11.7.1 with Chai 4.2.0 assertions
- **Frontend**: Jest (via react-scripts test)

### Build & Development Tools
- **Transpilation**: Babel 7.6.2 (Node.js current target)
- **Hot Reloading**: nodemon 3.1.10
- **Linting**: ESLint 8.57.0
  - Server: Airbnb Base configuration
  - Client: React/recommended configuration
- **Formatting**: Prettier 3.6.2
- **Git Hooks**: Husky 9.1.7 with lint-staged 16.1.2

### DevOps & Deployment
- **Containerization**: Docker with docker-compose 3.8
- **Orchestration**: Kubernetes (planned for Linode LKE)
- **Web Server**: nginx (reverse proxy and static serving)
- **Current Hosting**: Heroku (production)
- **CI/CD**: GitHub Actions
- **Container Registry**: Docker Hub
  - `maxjeffwell/educationelly-graphql-server`
  - `maxjeffwell/educationelly-graphql-client`
- **Security Scanning**: Trivy
- **Multi-Architecture**: linux/amd64 and linux/arm64

### Package Manager
- **npm**: Default package manager for both client and server

---

## Coding Standards

### Server-Side Rules (educationELLy-graphql-server)

#### Style Guide
- **Base Configuration**: Airbnb Base (airbnb-base)
- **ECMAScript Version**: ES2022
- **Module System**: ES Modules
- **Environment**: Node.js, ES2022, Mocha

#### Console Usage
- **Allowed**: `console.log`, `console.error`, `console.warn` for server-side logging
- Console statements are permitted for debugging and logging purposes

#### Naming Conventions
- **MongoDB IDs**: Allow underscore dangle for `_id` fields
- **Unused Variables**: Prefix with underscore (e.g., `_id`, `_context`) to indicate intentionally unused

#### Import/Export Rules
- **Default Exports**: Not required (`import/prefer-default-export: off`)
- **Dev Dependencies**: Relaxed rules for extraneous dependencies
- Supports flexible import patterns for GraphQL resolvers and schemas

#### Formatting Rules
- **Arrow Parens**: Off
- **Function Paren Newline**: Off
- **Comma Dangle**: Off
- **Trailing Spaces**: Warn
- **Multiple Empty Lines**: Warn
- **End of Line**: Warn

#### Test-Specific Rules
- **Unused Expressions**: Allowed (for Chai assertions like `expect(foo).to.be.true`)
- **Anonymous Functions**: Allowed in test files
- **Function Names**: Not required in test contexts

#### Relaxed Rules
The following rules are disabled for flexibility in GraphQL/MongoDB contexts:
- `implicit-arrow-linebreak`
- `no-return-await`
- `no-confusing-arrow`
- `consistent-return`
- `prefer-destructuring`

### Client-Side Rules (educationELLy-graphql-client)

#### Style Guide
- **Base Configuration**:
  - `eslint:recommended`
  - `plugin:react/recommended`
  - `plugin:react-hooks/recommended`
  - `plugin:jsx-a11y/recommended` (accessibility)
  - `plugin:import/recommended`
  - `prettier` (formatting)

#### ECMAScript & Environment
- **Version**: ES2021
- **Environments**: Browser, Node.js, Jest
- **Module System**: ES Modules

#### React-Specific Rules
- **JSX Transform**: React 17+ (no React import required in JSX scope)
- **Prop Types**: Warn level (runtime type checking encouraged)
- **Display Name**: Warn
- **Unescaped Entities**: Warn

#### React Hooks Rules
- **Rules of Hooks**: Error (must follow hooks rules)
- **Exhaustive Dependencies**: Warn (dependency array validation)

#### Import Order
- **Groups**: builtin → external → internal → parent → sibling → index
- **Newlines Between**: Always enforce newlines between import groups

#### Code Quality Rules
- **Unused Variables**: Warn
- **Console Statements**: Warn
- **Debugger**: Warn

#### Accessibility
- **Anchor Validity**: Warn (`jsx-a11y/anchor-is-valid`)

#### Prettier Integration
- Prettier formatting is enforced as an ESLint error
- All code must pass Prettier checks

#### Git Hooks (Pre-Commit)
Automated quality checks on commit:
1. ESLint auto-fix on staged `.js`/`.jsx` files
2. Prettier format on staged files
3. Block commit if linting fails

### Architecture Patterns

#### Separation of Concerns
- Clear separation between client and server codebases
- Each application has independent dependencies and configurations

#### GraphQL Architecture
- **Schema**: Type definitions split by domain (user, student)
- **Resolvers**: Include authorization logic and role-based access checks
- **Data Loading**: DataLoader prevents N+1 query problems through batching

#### Authentication & Authorization
- **Method**: JWT tokens via `x-token` HTTP header
- **Storage**: localStorage for client-side token persistence
- **Authorization**: Role-based access control implemented in resolvers
- **Error Handling**: Automatic sign-out on 401 errors

#### Component Organization
- Feature-based directory structure
- Components organized by functionality (App, Session, Students, etc.)

### Security Standards

#### Password Security
- **Hashing**: bcryptjs for secure password storage
- Never store plain text passwords

#### JWT Security
- **Secret Strength**: Use strong JWT secrets (64+ characters recommended)
- **Token Transmission**: Via HTTP headers only
- **Storage**: localStorage on client side

#### CORS Configuration
- Configured with credentials support
- Allowed origins specified via environment variables

#### Secrets Management
- **Never commit**: `.env` or `.env.production` files
- **Rotation**: Rotate secrets periodically
- **Environment Separation**: Use different credentials for dev and production

### Performance Best Practices

#### Data Loading
- **DataLoader**: Batch loading and caching for GraphQL resolvers
- **Query Caching**: Apollo InMemoryCache for client-side caching
- **Health Checks**: Implemented for monitoring and orchestration

---

## Project Structure

```
educationELLy-graphql/
├── CICD.md
├── docker-compose.prod.yml
├── docker-compose.yml
├── DOCKER.md
├── KUBERNETES.md
├── README.md
├── educationELLy-graphql-client/
│   ├── CLAUDE.md
│   ├── Dockerfile
│   ├── LICENSE
│   ├── nginx.conf
│   ├── package.json
│   ├── package-lock.json
│   ├── Procfile
│   ├── README.md
│   ├── static.json
│   ├── .eslintrc.js
│   ├── public/
│   │   ├── favicon.ico
│   │   ├── index.html
│   │   ├── manifest.json
│   │   ├── robots.txt
│   │   └── sitemap.xml
│   ├── screenshots/
│   │   ├── Screen Shot 2025-08-16 at 06.11.10.png
│   │   ├── Screen Shot 2025-08-16 at 06.11.19.png
│   │   ├── Screen Shot 2025-08-16 at 06.11.23.png
│   │   ├── Screen Shot 2025-08-16 at 06.13.07.png
│   │   ├── Screen Shot 2025-08-16 at 06.13.32.png
│   │   └── Screen Shot 2025-08-16 at 06.13.42.png
│   └── src/
│       ├── config.js
│       ├── index.js (Apollo Client setup, app entry point)
│       ├── components/
│       │   ├── App/ (main app component with routing)
│       │   ├── Session/ (authentication HOCs and session management)
│       │   ├── Students/ (student CRUD operations)
│       │   └── [Feature]/ (other feature components)
│       └── constants/
├── educationELLy-graphql-server/
│   ├── CLAUDE.md
│   ├── Dockerfile
│   ├── LICENSE
│   ├── package.json
│   ├── package-lock.json
│   ├── Procfile
│   ├── README.md
│   ├── seed.js
│   ├── server.log
│   ├── .eslintrc.js
│   ├── scripts/
│   │   ├── seed-production.js
│   │   └── seed-students-only.js
│   └── src/
│       ├── index.js (server entry point)
│       ├── loaders/ (DataLoader instances for batch loading)
│       ├── models/ (Mongoose models: User, Student)
│       ├── resolvers/ (GraphQL resolvers with authorization)
│       ├── schema/ (GraphQL type definitions by domain)
│       └── tests/
│           ├── api.js (API integration tests)
│           └── user.spec.js (user-specific tests)
├── k8s/
│   ├── client-deployment.yaml
│   ├── client-service.yaml
│   ├── configmap.yaml
│   ├── hpa.yaml (horizontal pod autoscaling)
│   ├── ingress.yaml
│   ├── namespace.yaml
│   ├── secrets.yaml.example
│   ├── server-deployment.yaml
│   └── server-service.yaml
└── nginx/
    └── nginx.conf (reverse proxy configuration)
```

### Directory Descriptions

#### Root Level
- **Documentation**: README.md, DOCKER.md, KUBERNETES.md, CICD.md
- **Docker Configs**: docker-compose.yml (dev), docker-compose.prod.yml (production)
- **Kubernetes**: k8s/ directory with complete deployment manifests
- **Nginx**: Reverse proxy configuration

#### Client Application (educationELLy-graphql-client/)
- **src/index.js**: Apollo Client setup and app entry point
- **src/components/**: Feature-based component organization
  - **App/**: Main app component with routing logic
  - **Session/**: Authentication HOCs and session management
  - **Students/**: Student CRUD operations
- **src/constants/**: Application constants
- **public/**: Static assets (HTML, manifest, icons)
- **Dockerfile**: Multi-stage build for production
- **nginx.conf**: Client-specific nginx configuration

#### Server Application (educationELLy-graphql-server/)
- **src/index.js**: Server entry point with Apollo Server setup
- **src/models/**: Mongoose models (User, Student)
- **src/schema/**: GraphQL type definitions split by domain
- **src/resolvers/**: GraphQL resolvers with authorization
- **src/loaders/**: DataLoader instances for batch loading
- **src/tests/**: API integration and unit tests
- **scripts/**: Database seeding scripts
- **Dockerfile**: Production-ready Node.js container

#### Kubernetes (k8s/)
Complete Kubernetes deployment configuration:
- **Deployments**: server-deployment.yaml, client-deployment.yaml
- **Services**: server-service.yaml, client-service.yaml
- **Configuration**: configmap.yaml, secrets.yaml.example
- **Networking**: ingress.yaml
- **Scaling**: hpa.yaml (horizontal pod autoscaler)
- **Namespace**: namespace.yaml

---

## External Resources

### Documentation

#### Project Documentation
- **README.md**: Main project documentation with quick start guide
- **DOCKER.md**: Detailed Docker and docker-compose deployment instructions
- **KUBERNETES.md**: Complete guide for deploying to Linode Kubernetes Engine
- **CICD.md**: GitHub Actions setup and workflow documentation

#### Application-Specific Documentation
- **educationELLy-graphql-server/README.md**: Backend API documentation and architecture
- **educationELLy-graphql-client/README.md**: Frontend application documentation
- **educationELLy-graphql-server/CLAUDE.md**: AI assistant guidance for server codebase
- **educationELLy-graphql-client/CLAUDE.md**: AI assistant guidance for client codebase

### Docker Images

#### Server Image
- **Name**: `maxjeffwell/educationelly-graphql-server`
- **Registry**: Docker Hub
- **Tags**: latest, version-specific
- **Architectures**: linux/amd64, linux/arm64
- **URL**: https://hub.docker.com/r/maxjeffwell/educationelly-graphql-server

#### Client Image
- **Name**: `maxjeffwell/educationelly-graphql-client`
- **Registry**: Docker Hub
- **Tags**: latest, version-specific
- **Architectures**: linux/amd64, linux/arm64
- **URL**: https://hub.docker.com/r/maxjeffwell/educationelly-graphql-client

### Production Deployments

#### Server
- **Platform**: Heroku
- **URL**: https://educationelly-server-graphql-5b9748151d5a.herokuapp.com
- **GraphQL Endpoint**: https://educationelly-server-graphql-5b9748151d5a.herokuapp.com/graphql

#### Client
- **Platform**: Heroku
- **URL**: https://educationelly-client-graphql-176ac5044d94.herokuapp.com

### External Services

#### Database
- **MongoDB**: External MongoDB instance
- **Connection**: Via `MONGODB_URI` environment variable
- **ORM**: Mongoose 8.16.3

#### CI/CD
- **Platform**: GitHub Actions
- **Workflows**: Continuous Integration, Docker Build and Push
- **Repository**: https://github.com/maxjeffwell/educationELLy-graphql

#### Code Coverage
- **Service**: Codecov
- **Integration**: Automatic coverage upload from CI pipeline

#### Security
- **Tool**: Trivy
- **Purpose**: Container vulnerability scanning in CI

### API Documentation

#### GraphQL API
- **Development**: http://localhost:8000/graphql
- **Production**: https://educationelly-server-graphql-5b9748151d5a.herokuapp.com/graphql
- **Explorer**: GraphQL Playground/Apollo Sandbox available

#### Health Check
- **Endpoint**: `/health`
- **Purpose**: Container and service health monitoring

### Third-Party Libraries

#### UI Components
- **Semantic UI React**: https://react.semantic-ui.com/
- **styled-components**: https://styled-components.com/

#### GraphQL
- **Apollo Server**: https://www.apollographql.com/docs/apollo-server/
- **Apollo Client**: https://www.apollographql.com/docs/react/

#### Database
- **Mongoose**: https://mongoosejs.com/
- **DataLoader**: https://github.com/graphql/dataloader

#### Authentication
- **jsonwebtoken**: https://github.com/auth0/node-jsonwebtoken
- **bcryptjs**: https://github.com/dcodeIO/bcrypt.js

### Development Tools

#### Linting & Formatting
- **ESLint**: https://eslint.org/
  - Server: Airbnb Base configuration
  - Client: React + Prettier configuration
- **Prettier**: https://prettier.io/

#### Git Hooks
- **Husky**: https://typicode.github.io/husky/
- **lint-staged**: https://github.com/lint-staged/lint-staged

---

## Additional Context

### Authentication Flow

#### Method
JWT-based authentication with role-based access control

#### Token Management
- **Header**: `x-token`
- **Storage**: localStorage (client-side)
- **Roles**: Admin, Teacher, Student
- **Error Handling**: Automatic sign-out on 401 errors

#### Implementation
1. User signs in with credentials
2. Server validates and returns JWT token
3. Client stores token in localStorage
4. Apollo Client automatically injects token in all requests
5. Server validates token and authorizes access based on roles

### Deployment Options

#### 1. Heroku (Current Production)
- **Status**: Active
- **Method**: Buildpacks with separate deployments
- **Server**: https://educationelly-server-graphql-5b9748151d5a.herokuapp.com
- **Client**: https://educationelly-client-graphql-176ac5044d94.herokuapp.com

#### 2. Docker (Available)
- **Status**: Ready for use
- **Method**: Containerized deployment with docker-compose
- **Environments**: Development and Production
- **Images**: Available on Docker Hub

#### 3. Kubernetes (Planned)
- **Status**: Manifests ready, deployment planned
- **Target**: Linode Kubernetes Engine (LKE)
- **Manifests**: Complete K8s configuration in `k8s/` directory

### CI/CD Pipeline

#### Continuous Integration
- **Platform**: GitHub Actions
- **Triggers**: Push and Pull Requests
- **Node Versions**: 18.x and 20.x
- **Steps**:
  1. Linting (ESLint)
  2. Code formatting checks (Prettier)
  3. Test execution with coverage
  4. Production build verification
  5. Coverage upload to Codecov

#### Docker Build and Push
- **Registry**: Docker Hub
- **Images**: Server and Client
- **Architectures**: linux/amd64, linux/arm64
- **Security**: Trivy vulnerability scanning

### Environment Configuration

#### Server Variables
- `MONGODB_URI`: MongoDB connection string
- `JWT_SECRET`: Secret for JWT signing (64+ characters recommended)
- `NODE_ENV`: Environment (development/production)
- `PORT`: Server port (default: 8000)
- `ALLOWED_ORIGINS`: CORS allowed origins (comma-separated)
- `TEST_DATABASE`: Database name for testing

#### Client Variables
- `REACT_APP_API_BASE_URL`: GraphQL API endpoint

### Recent Updates

#### Major Changes
1. Upgraded from Apollo Server 2.x to 4.x with new API patterns
2. Modernized dependency versions across both applications
3. Added comprehensive Kubernetes deployment manifests
4. Implemented multi-architecture Docker builds (amd64/arm64)
5. Added Trivy security scanning to CI pipeline
6. Fixed health check endpoints for container orchestration

### Technical Considerations

#### Testing
- **Server**: Tests require running server instance (two-terminal setup)
  - Terminal 1: `npm run test:run-server`
  - Terminal 2: `npm run test:execute-test`
- **Client**: Test infrastructure exists but no tests implemented yet

#### TypeScript
- TypeScript is NOT configured despite some references
- Project uses JavaScript (ES2022/ES2021)

#### Build Process
- Production builds skip ESLint checks for client
- Server uses Babel transpilation
- Client uses Create React App build pipeline

#### Database Isolation
- Test database isolation via `TEST_DATABASE` environment variable
- Separate databases for development, testing, and production

---

## Testing Instructions

### Server Testing

#### Setup
1. Start the test server in one terminal:
   ```bash
   cd educationELLy-graphql-server
   npm run test:run-server
   ```

2. Run tests in another terminal:
   ```bash
   cd educationELLy-graphql-server
   npm run test:execute-test
   ```

#### Test Structure
- **Framework**: Mocha + Chai
- **Location**: `src/tests/`
- **Files**:
  - `api.js`: API integration tests
  - `user.spec.js`: User-specific tests

#### Test Database
- Isolated test database via `TEST_DATABASE` environment variable
- Tests do not affect development or production data

### Client Testing

#### Running Tests
```bash
cd educationELLy-graphql-client
npm test
```

#### Test Configuration
- **Framework**: Jest (via react-scripts)
- **Options**:
  - `--env=jsdom`: Browser-like environment
  - `--passWithNoTests`: Don't fail if no tests exist

#### Current Status
- Test infrastructure is set up
- No tests are currently implemented
- Tests should be added for components, hooks, and utilities

### Quality Checks

#### Server Quality
```bash
cd educationELLy-graphql-server
npm run lint          # Run ESLint
npm run lint:fix      # Auto-fix linting issues
npm test              # Run tests
```

#### Client Quality
```bash
cd educationELLy-graphql-client
npm run lint          # Run ESLint (max-warnings 0)
npm run format        # Format with Prettier
npm run format:check  # Check formatting without changes
npm run quality       # Run all quality checks
npm test              # Run tests
```

---

## Build Steps

### Local Development Setup

#### Prerequisites
- Node.js 18+ installed
- MongoDB instance (local or remote)
- npm package manager

#### Server Setup
```bash
# Navigate to server directory
cd educationELLy-graphql-server

# Install dependencies
npm install

# Configure environment variables
cp .env.example .env
# Edit .env with your MongoDB URI and JWT secret

# Start development server (with hot reload)
npm run dev

# Server will be available at http://localhost:8000
# GraphQL playground at http://localhost:8000/graphql
```

#### Client Setup
```bash
# Navigate to client directory
cd educationELLy-graphql-client

# Install dependencies
npm install

# Configure environment variables
cp .env.example .env
# Edit .env with API endpoint (default: http://localhost:8000/graphql)

# Start development server
npm start

# Application will be available at http://localhost:3000
```

### Docker Development Setup

#### Using Docker Compose
```bash
# Start entire stack (server + client)
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

#### Server Only
```bash
docker-compose up -d server
```

#### Client Only
```bash
docker-compose up -d client
```

### Production Build

#### Server Production Build
```bash
cd educationELLy-graphql-server
npm install --production
npm start
```

#### Client Production Build
```bash
cd educationELLy-graphql-client
npm run build

# Serve static files (multiple options):
# 1. Using serve package
npx serve -s build

# 2. Using nginx (recommended for production)
# Copy build/ contents to nginx document root
```

### Docker Production Deployment

#### Build Images Locally
```bash
# Server
cd educationELLy-graphql-server
docker build -t educationelly-server .

# Client
cd educationELLy-graphql-client
docker build -t educationelly-client .
```

#### Using Production Compose
```bash
# Configure production environment
cp .env.production.example .env.production
# Edit .env.production with production values

# Start production stack
docker-compose -f docker-compose.prod.yml --env-file .env.production up -d
```

#### Using Pre-Built Images
```bash
# Pull from Docker Hub
docker pull maxjeffwell/educationelly-graphql-server:latest
docker pull maxjeffwell/educationelly-graphql-client:latest

# Run with docker-compose.yml
docker-compose up -d
```

### Kubernetes Deployment

#### Prerequisites
- Kubernetes cluster (Linode LKE or other)
- kubectl configured
- Docker images pushed to registry

#### Deploy to Kubernetes
```bash
# Create namespace
kubectl apply -f k8s/namespace.yaml

# Configure secrets (create from example)
cp k8s/secrets.yaml.example k8s/secrets.yaml
# Edit secrets.yaml with base64-encoded values

# Apply secrets
kubectl apply -f k8s/secrets.yaml

# Apply ConfigMap
kubectl apply -f k8s/configmap.yaml

# Deploy server
kubectl apply -f k8s/server-deployment.yaml
kubectl apply -f k8s/server-service.yaml

# Deploy client
kubectl apply -f k8s/client-deployment.yaml
kubectl apply -f k8s/client-service.yaml

# Configure ingress (optional)
kubectl apply -f k8s/ingress.yaml

# Enable autoscaling (optional)
kubectl apply -f k8s/hpa.yaml

# Check deployment status
kubectl get pods -n educationelly
kubectl get services -n educationelly
```

### Database Seeding

#### Development Data
```bash
cd educationELLy-graphql-server
node seed.js
```

#### Production Data
```bash
cd educationELLy-graphql-server
node scripts/seed-production.js
```

#### Students Only
```bash
cd educationELLy-graphql-server
node scripts/seed-students-only.js
```

---

## Data Models

### User Model
- **Fields**: username, email, password (hashed), role
- **Roles**: Admin, Teacher, Student
- **Authentication**: JWT tokens
- **Authorization**: Role-based access control

### Student Model
- **Fields**: Student profile and educational data
- **Relationships**: Associated with teachers and assessments
- **CRUD Operations**: Create, Read, Update, Delete via GraphQL

---

## GraphQL API

### Schema Organization
- Type definitions split by domain (user, student)
- Located in `src/schema/` directory
- Modular approach for maintainability

### Resolvers
- Located in `src/resolvers/` directory
- Include authorization logic
- Role-based access checks
- Error handling

### DataLoader
- Implemented in `src/loaders/` directory
- Prevents N+1 query problems
- Batches and caches database queries
- Improves performance significantly

---

## Routes

### Public Routes (Client)
- `/`: Home page
- `/signin`: User login
- `/signup`: User registration

### Protected Routes (Client)
- `/dashboard`: User dashboard (requires authentication)
- `/students`: Student list (requires authentication)
- `/student/*`: Individual student pages (requires authentication)

### API Endpoints (Server)
- `/graphql`: GraphQL API endpoint
- `/health`: Health check endpoint (for monitoring)

---

## Contributing Guidelines

### Code Quality
1. Each directory (client/server) has its own linting and formatting rules
2. Use the provided npm scripts for quality checks
3. Follow the existing code structure and patterns
4. Ensure all tests pass before submitting

### Git Workflow
1. Pre-commit hooks automatically run linting and formatting
2. Commits are blocked if linting fails
3. All staged files are automatically formatted

### Pull Requests
1. CI pipeline runs on all PRs
2. All quality checks must pass
3. Tests must pass on Node.js 18.x and 20.x
4. Docker builds must succeed

---

## Troubleshooting

### Common Issues

#### MongoDB Connection Failed
- Check `MONGODB_URI` in `.env` file
- Ensure MongoDB is running and accessible
- Verify network connectivity to MongoDB instance

#### JWT Authentication Errors
- Verify `JWT_SECRET` is configured in `.env`
- Ensure token is being sent in `x-token` header
- Check token expiration

#### CORS Errors
- Configure `ALLOWED_ORIGINS` in server `.env`
- Include client URL in allowed origins
- Ensure credentials are enabled

#### Docker Build Failures
- Check Dockerfile syntax
- Ensure all dependencies are properly installed
- Verify base image availability

#### Tests Failing
- Ensure test server is running (for server tests)
- Check `TEST_DATABASE` configuration
- Verify MongoDB test database is accessible

---

## Security Best Practices

### Secrets Management
1. Never commit `.env` or `.env.production` files
2. Use strong JWT secrets (64+ characters)
3. Configure CORS to only allow trusted origins
4. Keep MongoDB credentials secure
5. Rotate secrets periodically
6. Use different credentials for dev and production

### Code Security
1. All passwords are hashed with bcryptjs
2. JWT tokens are validated on every request
3. Role-based authorization prevents unauthorized access
4. Input validation on all user inputs
5. Parameterized queries prevent SQL injection
6. CORS configured with specific allowed origins

### Container Security
1. Multi-stage Docker builds minimize attack surface
2. Trivy scanning in CI pipeline
3. Regular dependency updates
4. Non-root user in containers
5. Health checks for monitoring

---

## Performance Optimization

### Backend
- DataLoader for batch loading and caching
- MongoDB indexes on frequently queried fields
- Connection pooling for database
- Efficient GraphQL resolvers

### Frontend
- Apollo Client InMemoryCache
- Code splitting with React Router
- Lazy loading of components
- Optimized production builds
- Static asset caching

### Infrastructure
- Horizontal Pod Autoscaling (HPA) in Kubernetes
- Health checks for load balancer routing
- Multi-architecture Docker images
- CDN for static assets (when applicable)

---

## Monitoring & Logging

### Health Checks
- Server: `/health` endpoint
- Client: `/health` endpoint (nginx)
- Kubernetes liveness and readiness probes

### Logging
- Server: Morgan HTTP request logging
- Console logging for debugging (development)
- Structured logging recommended for production

### Metrics
- CI/CD pipeline metrics via GitHub Actions
- Docker Hub image pull statistics
- Kubernetes pod metrics (when deployed)

---

## License

This project is licensed under the GNU General Public License v3.0 (GPLv3).

See the LICENSE files in the client and server directories for full license text.

---

## Support & Contact

**Author**: Jeff Maxwell
**Email**: maxjeffwell@gmail.com
**GitHub**: https://github.com/maxjeffwell/educationELLy-graphql

For issues, feature requests, or contributions, please use the GitHub repository issue tracker.

---

*This document is maintained for AI agents working with the educationELLy GraphQL project. Last updated: 2025-12-08*
