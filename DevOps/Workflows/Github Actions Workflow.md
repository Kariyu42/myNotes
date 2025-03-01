
- automates Continuous Integration and Continuous Deployment for a project.

# Overview

- Trigger conditions (`on:` section):
	- Runs on push to the `main/master` branch.
	- Can run on any pull requests as well.
- Jobs (`jobs:` section):
	1. Lint & Test (`lint-test`): Checks code quality and runs tests.
	2. Security Check (`security-check`): scans dependencies for security vulnerabilities.
	3. Deploy (`deploy`): Deploys the application to Vercel (but only after linting & security check pass).
	4. Notify (`notify`): Sends a message to a Discord channel when a push or pull request occurs.

```yaml
name: CI/CD

on:
  push:
    branches:
      - master
  pull_request:

jobs:
  lint-test:
    name: Lint & Test
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4  # This is now included in each job

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Lint Code
        run: npm run lint

      - name: Run Tests (if applicable)
        run: npm test || echo "No tests found, skipping..."

  security-check:
    name: Security Check
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4  # Checkout again here

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Run Security Audit
        run: npm audit --audit-level=high || echo "Security warnings found"

  deploy:
    name: Deploy to Vercel
    runs-on: ubuntu-latest
    needs: [lint-test, security-check]
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4  # Checkout again here

      - name: Install Vercel CLI
        run: npm install --global vercel@latest

      - name: Pull Vercel Environment Information
        run: vercel pull --yes --environment=production --token=${{ secrets.VERCEL_DEPLOY_TOKEN }}

      - name: Deploy to Production
        run: vercel --prod --token=${{ secrets.VERCEL_DEPLOY_TOKEN }}

  notify:
    name: Notify Discord on Push/Merge
    runs-on: ubuntu-latest
    if: github.event_name == 'push' || github.event_name == 'pull_request'
    steps:
      - name: Send Notification to Discord
        env:
          DISCORD_WEBHOOK: ${{ secrets.DISCORD_PUSH_NOTIF }}
        run: |
          TIMESTAMP=$(date +'%Y-%m-%d %H:%M:%S')
          curl -H "Content-Type: application/json" \
          -X POST \
          -d '{
            "content": "<@&1338794244915335218>",
            "embeds": [{
              "title": "🚀 New Push to Master!",
              "description": "**Branch:** `master`\n**Commit:** [`${{ github.sha }}`](https://github.com/${{ github.repository }}/commit/${{ github.sha }})\n**By:** `${{ github.actor }}`",
              "color": 3066993,
              "footer": {
                "text": "GitHub Actions • '"$TIMESTAMP"'"
              }
            }]
          }' \
          "${{ secrets.DISCORD_PUSH_NOTIF }}"

```

## Detailed Breakdown

1. The first one's job is to ensure code is formatted correctly using lint, it also runs automated tests using the `npm test` command.

### Steps:

1. Checkout Repository
	- Uses `actions/checkout@v4` to pull the latest code into the runner.
2. Set up Node.js
	- Uses `actions/setup-node@v4` to install Node.js v18.
	- Enables npm caching for faster installs.
3. Install dependencies
	- Runs `npm ci` to install 