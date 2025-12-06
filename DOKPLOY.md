# Dokploy Deployment Guide

This guide will help you deploy the Quran Hub API on Dokploy.

## Prerequisites

- A Dokploy instance running and accessible
- Git repository with this codebase
- Access to configure environment variables in Dokploy

## Deployment Steps

### 1. Prepare Your Repository

Ensure your repository contains:
- `docker-compose.yml` ✅
- `Dockerfile` ✅
- `requirements.txt` ✅
- `database/quranhub_snapshot_dump.sql` ✅

### 2. Create a New Application in Dokploy

1. Log in to your Dokploy dashboard
2. Navigate to **Applications** → **New Application**
3. Select **Docker Compose** as the deployment method
4. Connect your Git repository (GitHub, GitLab, or Bitbucket)

### 3. Configure Environment Variables

In Dokploy's environment variables section, set the following:

```env
DB_USERNAME=quranhub
DB_PASSWORD=your_secure_password_here
DB_HOST=postgres
DB_PORT=5432
DB_NAME=quranhub
APP_PORT=8080
```

**Important:** 
- Use a strong password for `DB_PASSWORD`
- The `DB_HOST` should remain as `postgres` (the service name in docker-compose)
- Dokploy will automatically set up the `DATABASE_URL` if needed

### 4. Configure Domain (Optional)

1. In Dokploy, go to your application settings
2. Navigate to **Domains**
3. Add your custom domain (e.g., `api.quranhub.com`)
4. Dokploy will automatically configure SSL certificates via Let's Encrypt

### 5. Deploy

1. Click **Deploy** in Dokploy
2. Dokploy will:
   - Clone your repository
   - Build the Docker images
   - Start PostgreSQL and load the database dump
   - Start the application
   - Set up reverse proxy

### 6. Verify Deployment

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
- The database dump (`quranhub_snapshot_dump.sql`) is automatically loaded on first deployment
- This happens only once when the PostgreSQL container is first initialized

### Resetting the Database
If you need to reload the database:

1. In Dokploy, stop the application
2. Remove the PostgreSQL volume (via Dokploy's volume management or SSH)
3. Redeploy the application

### Accessing the Database
- The PostgreSQL service is internal to the Docker network
- To access it externally, you may need to expose the port or use Dokploy's database management features

## Troubleshooting

### Application Won't Start
- Check application logs in Dokploy
- Verify environment variables are set correctly
- Ensure PostgreSQL health check is passing

### Database Connection Issues

#### Password Authentication Failed Error
If you see `password authentication failed for user "quranhub"`, this usually means:

1. **Database was initialized with different credentials**: The PostgreSQL volume already exists with old credentials
   - **Solution**: Remove the PostgreSQL volume and redeploy
   - In Dokploy: Stop the application → Go to Volumes → Delete `postgres_data` volume → Redeploy
   - Or via SSH: `docker volume rm quranhub_postgres_data` (adjust name as needed)

2. **Environment variables mismatch**: The credentials in Dokploy don't match what PostgreSQL expects
   - **Solution**: Ensure all environment variables are set correctly:
     - `DB_USERNAME` must match `POSTGRES_USER` in the postgres service
     - `DB_PASSWORD` must match `POSTGRES_PASSWORD` in the postgres service
     - Both services use the same values from environment variables

3. **Special characters in password**: If your password contains special characters, they may need URL encoding
   - **Solution**: Use a password without special characters, or ensure proper URL encoding in `DATABASE_URL`

#### Other Database Issues
- Verify `DB_HOST=postgres` (must match service name)
- Check that PostgreSQL container is healthy
- Review database logs in Dokploy
- Ensure `DATABASE_URL` is correctly formatted: `postgresql://username:password@host:port/database`

### Port Conflicts
- Dokploy handles port mapping automatically via reverse proxy
- If issues occur, check the `APP_PORT` environment variable

## Environment Variables Reference

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `DB_USERNAME` | PostgreSQL username | `quranhub` | No |
| `DB_PASSWORD` | PostgreSQL password | - | **Yes** |
| `DB_HOST` | Database host (service name) | `postgres` | No |
| `DB_PORT` | Database port | `5432` | No |
| `DB_NAME` | Database name | `quranhub` | No |
| `APP_PORT` | Application port | `8080` | No |

## Support

For Dokploy-specific issues, refer to:
- [Dokploy Documentation](https://docs.dokploy.com)
- [Dokploy Docker Compose Guide](https://docs.dokploy.com/docs/core/docker-compose)

