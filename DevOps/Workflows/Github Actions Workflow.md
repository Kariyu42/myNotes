
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

```