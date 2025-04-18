# Jenkins CI/CD pipeline
This Jenkins pipeline script automates a series of tasks in a CI/CD workflow, specifically for building and deploying a Docker-based Node.js application, with different behaviors based on the Git branch.

Logo Selection: Based on the Git branch (main or dev), the appropriate logo (logo-main.svg or logo-dev.svg) is copied to the application source directory.

Lint Dockerfile: The pipeline uses a custom dockerLint() function to lint the Dockerfile for potential issues or improvements.

Build: It installs Node.js dependencies using npm install to prepare the application for testing and packaging.

Test: Executes npm test to run unit tests and ensure the code works as expected before building the Docker image.

Build and Scan Docker Image:

Based on the Git branch, it tags the Docker image (e.g., nodemain-v1.0 for the main branch or nodedev-v1.0 for the dev branch).

The image is then built and scanned for vulnerabilities using trivy, reporting any critical issues.

After a successful build, it pushes the image to Docker Hub.

Additionally, based on the branch, the pipeline triggers the respective deployment job (Deploy_to_main or Deploy_to_dev).
