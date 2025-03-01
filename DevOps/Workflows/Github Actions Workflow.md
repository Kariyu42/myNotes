
- automates Continuous Integration and Continuous Deployment for a project.

# Overview

- Trigger conditions (`on:` section):
	- Runs on push to the `main/master` branch.
	- Can run on any pull requests as well.
- Jobs (`jobs:` section):
	1.Lint & Test (`lint-test`): Checks code 