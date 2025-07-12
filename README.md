# Docker-CGWire

Docker compose for [Kitsu](https://kitsu.cg-wire.com/) and [Zou](https://zou.cg-wire.com/)

## Notice

This project is a detached version of [MathieuBOUZARD's](https://gitlab.com/mathbou) cgwire automation project on [gitlab](https://gitlab.com/mathbou/docker-cgwire). Please make sure to check out that project as well.

## About CGWire

CGWire is an open-source project management tool designed for animation and VFX studios. It consists of two main components:

- **Kitsu**: A web-based project management tool for tracking animation production
- **Zou**: The API server and database backend for Kitsu

## Changes for eefafx

We are using the same flow and same docker containers as the original project, with some minor changes to make the deployment process easier:

- Reading environment variables from `.env` file
- Sourcing the `.env` file at the start of scripts
- Added a `.env.sample` file to follow along easily
- Improved documentation and deployment scripts

## Prerequisites

Before running this project, make sure you have the following installed:

- Docker and Docker Compose
- `curl` (for fetching version information)
- `jq` (for JSON parsing)
- `git` (for cloning repositories)

## Quick Start

### 1. Setup Environment

First, copy the sample environment file and customize it:

```bash
cp .env.sample .env
```

Edit `.env` file with your specific configuration (database passwords, mail settings, etc.).

### 2. Run the Stack

Make the build script executable and run it:

```bash
chmod +x build.sh
./build.sh local
```

This will start the following docker containers:

```
<id> kitsu:0.20.69                 "/docker-entrypoint.…"   6 days ago   Up 6 days   0.0.0.0:80->80/tcp, tcp           eefa-kitsu
<id> zou:0.20.57                   "gunicorn --error-lo…"   6 days ago   Up 6 days                                                 eefa-zou-event
<id> zou:0.20.57                   "gunicorn --error-lo…"   6 days ago   Up 6 days   0.0.0.0:5000->5000/tcp, eefa-zou-app
<id> postgres:15-alpine            "docker-entrypoint.s…"   6 days ago   Up 6 days   5432->5432/tcp, [::]:5432->5432/tcp   eefa-db-15
<id> redis:alpine                  "docker-entrypoint.s…"   6 days ago   Up 6 days   6379/tcp                                      eefa-redis
<id> zou:0.20.57                   "rq worker -c zou.jo…"   6 days ago   Up 6 days                                                 eefa-zou-jobs
<id> getmeili/meilisearch:v1.1.1   "tini -- /bin/sh -c …"   ago   Up 6 days   7700/tcp                                      eefa-indexer
```

### 3. Access the Application

Once the containers are running, you can access:

- **Kitsu Web Interface**: http://localhost (or your configured PORT)
- **Zou API**: http://localhost:5000 (internal)
- **PostgreSQL Database**: localhost:5432 (internal)
- **Redis**: localhost:6379 (internal)
- **Meilisearch**: localhost:7700 (internal)

## Usage

### Build Script Commands

The main `build.sh` script supports the following commands:

#### Basic Commands

```bash
# Build and start the stack locally
./build.sh local

# Stop and remove containers
./build.sh down
```

#### Development Mode

```bash
# Start in development mode (clean database on each restart)
./build.sh local --develop

# Development mode with persistent database
./build.sh local --develop --keep-db

# Stop development stack
./build.sh down --develop
```

#### Custom Environment

```bash
# Use a custom environment file
./build.sh local --env=/path/to/custom.env

# Or set the environment variable
ENV_FILE=/path/to/custom.env ./build.sh local
```

#### Help

```bash
# Show help information
./build.sh --help
```

### Database Management

#### Database Upgrade

Use the `db_upgrade.sh` script to upgrade your database between versions:

```bash
# Upgrade database from version X to version Y
./db_upgrade.sh 14 15

# Dry run (test without making changes)
./db_upgrade.sh --dry-run 14 15

# Use custom environment file
./db_upgrade.sh --env=/path/to/custom.env 14 15
```

#### LDAP Synchronization

If you're using LDAP authentication, use the `sync_ldap.sh` script:

```bash
# Synchronize LDAP users
./sync_ldap.sh

# Use custom environment file
./sync_ldap.sh --env=/path/to/custom.env
```

## Configuration

### Environment Variables

The project uses environment variables for configuration. Key variables include:

#### Project Settings
- `COMPOSE_PROJECT_NAME`: Name prefix for Docker containers (default: `eefa`)
- `DEBUG`: Enable debug mode (default: `True`)
- `PORT`: Port for Kitsu web interface (default: `80`)

#### Version Control
- `ZOU_VERSION`: Zou API version (default: `0.20.57`)
- `KITSU_VERSION`: Kitsu frontend version (default: `0.20.69`)

#### Database Configuration
- `DB_HOST`: Database hostname (default: `db`)
- `DB_PORT`: Database port (default: `5432`)
- `DB_VERSION`: PostgreSQL version (default: `15`)
- `DB_PASSWORD`: Database password
- `DB_POOL_SIZE`: Connection pool size (default: `30`)
- `DB_MAX_OVERFLOW`: Maximum overflow connections (default: `60`)

#### Redis Configuration
- `KV_HOST`: Redis hostname (default: `redis`)
- `KV_PORT`: Redis port (default: `6379`)

#### Job Queue
- `ENABLE_JOB_QUEUE`: Enable background job processing (default: `True`)
- `PREVIEW_FOLDER`: Directory for preview files (default: `/opt/zou/previews`)
- `TMP_DIR`: Temporary directory (default: `/tmp/zou`)

#### Event Stream
- `EVENT_STREAM_HOST`: Event stream hostname (default: `zou-event`)

#### Search Index
- `INDEXER_VERSION`: Meilisearch version (default: `v1.1.1`)
- `INDEXER_KEY`: Meilisearch API key
- `INDEXER_HOST`: Meilisearch hostname (default: `indexer`)

#### User Management
- `USER_LIMIT`: Maximum number of users (default: `200`)

#### Email Configuration
- `MAIL_SERVER`: SMTP server hostname
- `MAIL_PORT`: SMTP server port
- `MAIL_USERNAME`: SMTP username
- `MAIL_PASSWORD`: SMTP password
- `MAIL_DEBUG`: Enable email debug mode (default: `0`)
- `MAIL_USE_TLS`: Use TLS for email (default: `False`)
- `MAIL_USE_SSL`: Use SSL for email (default: `True`)
- `MAIL_DEFAULT_SENDER`: Default sender email address
- `DOMAIN_NAME`: Your domain name
- `DOMAIN_PROTOCOL`: Protocol (http/https)

### Docker Compose Files

The project includes several Docker Compose files for different use cases:

- `docker-compose.yml`: Main production configuration
- `docker-compose.develop.yml`: Development mode with volume mounts
- `docker-compose.build.yml`: Local build configuration
- `docker-compose.prod.yml`: Production-specific settings
- `docker-compose.dbUpgrade.yml`: Database upgrade configuration

## Architecture

The CGWire stack consists of several interconnected services:

### Services Overview

1. **Kitsu (Frontend)**
   - Nginx-based web server
   - Serves the Vue.js frontend application
   - Accessible on port 80 (configurable)

2. **Zou-App (API Server)**
   - Python Flask application
   - Main API backend
   - Handles HTTP requests and business logic
   - Accessible on port 5000

3. **Zou-Event (Event Stream)**
   - WebSocket server for real-time updates
   - Handles live notifications and updates
   - Internal service, not directly accessible

4. **Zou-Jobs (Background Jobs)**
   - RQ (Redis Queue) worker
   - Processes background tasks
   - Handles file processing, notifications, etc.

5. **Database (PostgreSQL)**
   - Stores all application data
   - Persistent storage with volumes
   - Accessible on port 5432

6. **Redis**
   - Key-value store for caching
   - Message broker for job queue
   - Session storage

7. **Meilisearch (Indexer)**
   - Full-text search engine
   - Indexes project data for fast searching
   - Internal service

### Data Persistence

The following volumes are used for data persistence:

- `previews`: Stores preview files and thumbnails
- `tmp`: Temporary files and processing
- Database data is persisted in Docker volumes

## Troubleshooting

### Common Issues

#### Port Conflicts
If you encounter port conflicts, modify the `PORT` variable in your `.env` file:

```bash
PORT=8080  # Change from default 80
```

#### Database Connection Issues
Ensure the database is fully initialized before the API starts:

```bash
# Check database logs
docker logs eefa-db-15

# Restart the stack
./build.sh down
./build.sh local
```

#### Permission Issues
Make sure the build script is executable:

```bash
chmod +x build.sh
chmod +x db_upgrade.sh
chmod +x sync_ldap.sh
```

#### Memory Issues
If you encounter memory issues, consider:

1. Reducing `DB_POOL_SIZE` and `DB_MAX_OVERFLOW`
2. Increasing Docker memory limits
3. Using production mode instead of development mode

### Log Access

View logs for specific services:

```bash
# View all logs
docker-compose logs

# View logs for specific service
docker-compose logs zou-app
docker-compose logs kitsu
docker-compose logs db

# Follow logs in real-time
docker-compose logs -f zou-app
```

### Development Mode

Development mode provides additional features for developers:

- Direct access to source code through volume mounts
- Hot reload capabilities
- Easier debugging and testing
- Clean database on each restart (unless `--keep-db` is used)

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project maintains the same license as the original CGWire components. Please refer to the individual LICENSE files in the `kitsu/` and `zou/` directories.

## Support

For issues specific to this Docker setup, please open an issue in this repository.

For CGWire-specific issues, please refer to:
- [Kitsu Documentation](https://kitsu.cg-wire.com/)
- [Zou Documentation](https://zou.cg-wire.com/)
- [CGWire Community](https://discord.gg/VbCxtKN)
