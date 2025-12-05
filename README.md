# DrupalQuickSetup

## Overview
This repository provides a complete, reproducible Drupal 10 local environment using Docker. It includes:

- Drupal 10 + Apache  
- PostgreSQL 15  
- Persistent user-uploaded files  
- Persisted Drupal configuration (`settings.php` and `services.yml`)  
- JSON:API enabled and CORS configured for external applications (such as React frontends)

This setup is designed as a proof-of-concept CMS backend for embedding Drupal-managed pages into external applications.

---

## Requirements
- Docker Desktop (Windows, macOS, or Linux)

No local PHP, Composer, Drush, or database installation is required.

---

## Quick Start

### 1. Clone the repository
```bash
git clone https://github.com/<your-org>/DrupalQuickSetup.git
cd DrupalQuickSetup
```
### 2. Make sure Docker Desktop is running

### 3. Start the containers
```bash
docker compose up -d
```

### 3. Access Drupal

Drupal UI: http://localhost:8080

JSON:API: http://localhost:8080/jsonapi

If this is your first run, Drupal will show the installation page.

Use these database settings during install:

Database Configuration
- Host: db
- Database name: drupal
- Database user: drupal
- Database password: drupal

Create any admin username/password you want. All data persists automatically.

### 4. How Persistence Works

This setup mounts three persistent volumes/directories:

- `db_data` → PostgreSQL database files  
- `drupal_data` → Entire Drupal installation (core + modules)  
- `./drupal-config/sites/default/` → Drupal configuration overrides  
- `./drupal-files/` → The Drupal `/sites/default/files` directory  

This means:
- Your settings.php is preserved  
- Your services.yml (CORS config) is preserved  
- Uploaded images and files persist  
- Rebuilding the container does NOT wipe Drupal  

---

### 5. Testing Persistence

To confirm your setup is correct:

1. Create a page in Drupal.
2. Upload an image.
3. Edit CORS settings in `services.yml`.
4. Run: `docker compose down`
5. Run: `docker compose up -d`
6. Visit Drupal again → Your page, files, and settings should still exist.

If everything is still working, the persistence system is correct.

---

### 6. Resetting Drupal Completely (optional)

If you ever want a clean slate:

```bash
docker compose down -v
```


This removes database + Drupal volumes.  
Your mounted config `settings.php`, `services.yml`, and `drupal-files` remain.

---

### 8. Troubleshooting

**CORS blocked when requesting JSON:API**

Edit `drupal-config/sites/default/services.yml`:

## CORS Configuration (services.yml)

Make sure your `services.yml` contains the following:

```yaml
cors.config:
  enabled: true

  # Allow your frontend access
  allowedOrigins: ['http://localhost:3000'] # Or whatever port you run your frontend on

  # Allow all headers
  allowedHeaders: ['*']

  # Allow all HTTP methods (GET, POST, PATCH, DELETE, etc.)
  allowedMethods: ['*']

  # Credentials not required for basic JSON:API access
  supportsCredentials: false
```

## Verify CORS Is Working

Run this from Windows PowerShell:

``` bash
C:\Windows\System32\curl.exe -I http://localhost:8080/jsonapi
```

You should receive an HTTP/1.1 200 OK response.  

If CORS is active, you will also see a header like:

You should see:

``` yaml
Vary: Origin
```

If this header appears, Drupal is correctly applying your CORS configuration and your frontend (http://localhost:3000) is allowed to make requests.

If you do NOT see this header or your frontend still receives CORS errors, rebuild containers:

``` bash
docker compose restart
```

or fully recreate them:

``` bash
docker compose down
docker compose up -d
```

---------------------------------------
## Troubleshooting

### 1. CORS Still Not Working
Make sure your mounted files match the paths in docker-compose:

``` yaml
drupal-config/sites/default/settings.php → /var/www/html/sites/default/settings.php
drupal-config/sites/default/services.yml  → /var/www/html/sites/default/services.yml
```

Then restart Drupal:

``` bash
docker compose restart drupal
```

### 2. Verify File Mounts Inside the Container
Enter the Drupal container:

``` bash
docker exec -it drupal-drupal-1 bash
```

Then check:

``` yaml
ls -l /var/www/html/sites/default/settings.php
ls -l /var/www/html/sites/default/services.yml
```

Both files should show as coming from your local machine (“bind mount”).

### 3. Check the CORS Block in services.yml

It should look exactly like this:

``` yaml
cors.config:
  enabled: true
  allowedOrigins: ['http://localhost:3000']
  allowedHeaders: ['*']
  allowedMethods: ['*']
  supportsCredentials: false
```

### 4. Browser Caching Issues
Browsers heavily cache CORS failures.

Hard refresh or restart the browser.  
If still failing: open DevTools → Network → disable cache.

---------------------------------------
## Persisted Project Structure

``` yaml
DrupalQuickSetup/
│
├── docker-compose.yaml
│
├── drupal-config/
│   └── sites/
│       └── default/
│           ├── settings.php
│           └── services.yml
│
└── drupal-files/
    └── (populated automatically by Drupal)
```

---------------------------------------
## Resetting to a Clean Slate

Remove all containers **and volumes**:

``` bash
docker compose down -v
```

Recreate everything:

``` bash
docker compose up -d
```

Then visit:

http://localhost:8080

Run through Drupal’s installer again using:

``` yaml
Database host: db  
Database name: drupal  
Database user: drupal  
Database password: drupal  
```

Your custom CORS settings will still load automatically.

---------------------------------------
## Summary

This setup ensures:

• Drupal configuration persists  
• Drupal file uploads persist  
• CORS is always enabled for your frontend  
• Reinstalling Drupal does not require reapplying settings  
• Anyone can clone the repo and run the stack immediately  

This completes the reproducible Drupal 10 quick-setup environment.
