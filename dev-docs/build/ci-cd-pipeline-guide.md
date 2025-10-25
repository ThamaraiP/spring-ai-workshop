# CI/CD Pipeline Guide

Our Jenkins pipeline includes:
1. **Build:** `mvn clean install`
2. **Test:** Integration tests
3. **Deploy:** Publish Docker image to registry

Artifacts are stored at `/var/lib/jenkins/workspace/builds`.
Common variables:
- `BUILD_ID`
- `GIT_COMMIT`
- `JOB_NAME`
