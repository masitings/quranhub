# Dokploy Deployment Guide

This guide will help you deploy the Quran Hub API on Dokploy.

## Prerequisites

- A Dokploy instance running and accessible
- Git repository with this codebase
- Access to configure environment variables in Dokploy
- **External PostgreSQL database** (the docker-compose file does not include a PostgreSQL service)

## Deployment Steps

### 1. Prepare Your Database

**Important:** This application requires an external PostgreSQL database. You need to:

1. Set up a PostgreSQL database (can be on Dokploy, cloud provider, or external server)
2. Load the database dump (`database/quranhub_snapshot_dump.sql`) into your PostgreSQL instance
3. Note the database connection details (host, port, username, password, database name)

### 2. Prepare Your Repository

Ensure your repository contains:
- `docker-compose.yml` ✅
- `Dockerfile` ✅
- `requirements.txt` ✅

### 3. Create a New Application in Dokploy

1. Log in to your Dokploy dashboard
2. Navigate to **Applications** → **New Application**
3. Select **Docker Compose** as the deployment method
4. Connect your Git repository (GitHub, GitLab, or Bitbucket)

### 4. Configure Environment Variables

In Dokploy's environment variables section, set the following:

```env
DB_USERNAME=your_database_username
DB_PASSWORD=your_database_password
DB_HOST=your_database_host
DB_PORT=5432
DB_NAME=your_database_name
APP_PORT=8080
```

**Important:** 
- `DB_HOST` should be the hostname or IP address of your external PostgreSQL database
  - If using Dokploy's database service, use the service name or connection string provided
  - If using a cloud database (AWS RDS, DigitalOcean, etc.), use the provided hostname
  - If using a self-hosted database, use the server's IP or domain name
- `DB_PORT` is typically `5432` for PostgreSQL
- Ensure your database is accessible from the Dokploy network
- The `DATABASE_URL` will be automatically constructed from these variables

### 5. Configure Domain (Optional)

1. In Dokploy, go to your application settings
2. Navigate to **Domains**
3. Add your custom domain (e.g., `api.quranhub.com`)
4. Dokploy will automatically configure SSL certificates via Let's Encrypt

### 6. Deploy

1. Click **Deploy** in Dokploy
2. Dokploy will:
   - Clone your repository
   - Build the Docker image
   - Start the application
   - Set up reverse proxy

### 7. Verify Deployment

Once deployed, check:

1. **Health Endpoints:**
   - `https://your-domain.com/health/liveness`
   - `https://your-domain.com/health/readiness`
   - `https://your-domain.com/health/startup`

2. **API Documentation:**
   - `https://your-domain.com/docs` (Swagger UI)
   - `https://your-domain.com/redoc` (ReDoc)

3. **Test Endpoint:**
   - `https://your-domain.com/` (should return `{"message":"Success"}`)

## Auto-Deployment (Optional)

To enable automatic deployments on code changes:

1. In Dokploy, go to your application settings
2. Navigate to **Auto Deploy**
3. Configure webhook from your Git provider (GitHub, GitLab, Bitbucket)
4. Dokploy will automatically redeploy when you push to the configured branch

## Database Management

### Initial Setup
- You need to manually load the database dump (`database/quranhub_snapshot_dump.sql`) into your external PostgreSQL database
- Use `psql` or a database management tool to import the dump:
  ```bash
  psql -h your_database_host -U your_username -d your_database_name -f database/quranhub_snapshot_dump.sql
  ```

### Database Connection
- Ensure your external database is accessible from the Dokploy network
- Check firewall rules and network security groups
- For cloud databases, you may need to whitelist Dokploy's IP addresses

## Troubleshooting

### Application Won't Start
- Check application logs in Dokploy
- Verify environment variables are set correctly
- Ensure PostgreSQL health check is passing

### Database Connection Issues

#### Password Authentication Failed Error
If you see `password authentication failed for user "quranhub"`, this usually means:

1. **Incorrect credentials**: The database credentials don't match
   - **Solution**: Verify your `DB_USERNAME` and `DB_PASSWORD` environment variables match your external database
   - Double-check the credentials in your database management interface

2. **Database host unreachable**: The application cannot connect to the database host
   - **Solution**: 
     - Verify `DB_HOST` is correct (hostname or IP address)
     - Check network connectivity from Dokploy to your database
     - Ensure firewall rules allow connections from Dokploy's network
     - For cloud databases, verify security groups allow inbound connections on port 5432

3. **Special characters in password**: If your password contains special characters, they may need URL encoding
   - **Solution**: Use a password without special characters, or ensure proper URL encoding in `DATABASE_URL`

#### Other Database Issues
- Verify `DB_HOST` points to your external database hostname/IP
- Check that your external PostgreSQL database is running and accessible
- Review database connection logs in Dokploy
- Ensure `DATABASE_URL` is correctly formatted: `postgresql://username:password@host:port/database`
- Test database connectivity using `psql` or a database client from the same network

### Port Conflicts
- Dokploy handles port mapping automatically via reverse proxy
- If issues occur, check the `APP_PORT` environment variable

## Environment Variables Reference

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `DB_USERNAME` | PostgreSQL username | `quranhub` | **Yes** |
| `DB_PASSWORD` | PostgreSQL password | - | **Yes** |
| `DB_HOST` | External database hostname or IP | `localhost` | **Yes** |
| `DB_PORT` | Database port | `5432` | No |
| `DB_NAME` | Database name | `quranhub` | **Yes** |
| `APP_PORT` | Application port | `8080` | No |

## Support

For Dokploy-specific issues, refer to:
- [Dokploy Documentation](https://docs.dokploy.com)
- [Dokploy Docker Compose Guide](https://docs.dokploy.com/docs/core/docker-compose)

