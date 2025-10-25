# Jenkins Troubleshooting

| Issue | Cause | Fix |
|-------|--------|-----|
| Build fails with code 137 | Out of memory | Increase container memory limit |
| Git checkout errors | Wrong credentials | Reconfigure Git credentials |
| Stage skipped | Conditional stage not met | Check Jenkinsfile for `when` conditions |
| Docker build hangs | Docker daemon not reachable | Restart Docker service |
