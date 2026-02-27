# gitlab-setup

A simple and production-ready GitLab CE setup using Docker Compose with SSL support and GitLab Runner.

## Requirements

* Ubuntu 22.04 / 24.04
* Docker + Docker Compose
* A domain pointing to your server
* SSL certificate files (`fullchain.pem` and `privkey.pem`)

## Directory Structure

```
gitlab-project/
├── docker-compose.yml
├── .env
├── ssl/
│   ├── fullchain.pem
│   └── privkey.pem
├── gitlab/
│   ├── config/
│   ├── logs/
│   └── data/
└── runner/
    └── config/
```

## Setup

**1. Create the required directories**

```bash
mkdir -p ssl gitlab/config gitlab/logs gitlab/data runner/config
```

**2. Copy your SSL certificate files**

```bash
cp /path/to/fullchain.pem ssl/
cp /path/to/privkey.pem ssl/
```

**3. Configure environment variables**

```bash
cp .env.example .env
nano .env
```

Fill in your domain and SMTP settings inside `.env`.

**4. Start the services**

```bash
docker compose up -d
```

GitLab takes 3 to 5 minutes to fully start on the first run.

**5. Get the initial root password**

```bash
docker exec -it gitlab grep 'Password:' /etc/gitlab/initial_root_password
```

This file is automatically deleted after 24 hours. Save the password immediately.

**6. Open your browser**

Go to `https://your-domain.com` and log in with username `root` and the password from the previous step.

## Adding a GitLab Runner

**1. Get a runner token**

In GitLab, go to Admin Area -> CI/CD -> Runners -> New instance runner, fill in the form and click Create runner. Copy the token shown on the next page.

**2. Register the runner**

```bash
docker exec -it gitlab-runner gitlab-runner register \
  --non-interactive \
  --url "https://your-domain.com" \
  --token "YOUR_RUNNER_TOKEN" \
  --executor "docker" \
  --docker-image "alpine:latest" \
  --description "docker-runner"
```

**3. Verify**

Go to Admin Area -> CI/CD -> Runners and confirm the runner status is green.

## Security Recommendations

* Disable public sign-up: Admin Area -> Settings -> General -> Sign-up restrictions
* Set default project visibility to Private
* Enable two-factor authentication for all users
* Enable Admin Mode: Admin Area -> Settings -> General -> Sign-in restrictions
* Set a strong password for the root account and enable 2FA on it

## Backup

Run a manual backup at any time:

```bash
docker exec -it gitlab gitlab-backup create
```

Backups are stored in `./gitlab/data/backups/` and kept for 7 days by default.
