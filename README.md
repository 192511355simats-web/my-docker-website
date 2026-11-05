# RAMPRASADH Portfolio CI/CD

This repository builds the portfolio website into an Nginx Docker image. Jenkins checks for changes on GitHub, verifies the container serves the site, and deploys it on the machine running Docker at <http://localhost:8081>.

## How an edit reaches the website

1. Edit `index.html` or `style.css` in VS Code and save.
2. Commit and push the change to the GitHub branch configured in the Jenkins job (normally `main`). Saving a file in VS Code alone does not update GitHub.
3. Jenkins polls GitHub every two minutes. When it finds a new commit, it checks out the code, builds and checks the Docker image, then replaces the running site container. A scheduled build also runs every 15 minutes.
4. Refresh <http://localhost:8081> after the Jenkins build succeeds.

The Jenkins job polls SCM because Jenkins is running locally and GitHub cannot reach it directly. The schedule follows the lab reference's **Build periodically** procedure.

## Prerequisites

- Docker Desktop running with the Linux container engine.
- A Jenkins Linux container/agent with Git and Docker CLI installed, and access to the Docker Engine socket. The Jenkins build log should show the agent running on Linux.
- The public GitHub repository `https://github.com/192511355simats-web/my-docker-website`.

## Configure the Jenkins Pipeline job

1. Open Jenkins at <http://localhost:8080> and sign in.
2. Create a **Pipeline** job (or update the existing job) with **Pipeline script from SCM**.
3. Set **SCM** to **Git**, **Repository URL** to `https://github.com/192511355simats-web/my-docker-website.git`, **Branch Specifier** to `*/main`, and **Script Path** to `Jenkinsfile`.
4. Turn off **Lightweight checkout** so Jenkins uses a normal Git checkout.
5. Under **Build periodically**, set `H/15 * * * *`. Under **Poll SCM**, set `H/2 * * * *`.
6. Save and choose **Build Now**. The pipeline trusts only the current workspace path in Git before checkout, then builds, tests, and deploys the site.

The Jenkins agent must have permission to use Docker. The `ramprasadh-site` container publishes port `8081` on the Docker host; keep Docker Desktop and Jenkins running while using the site.
