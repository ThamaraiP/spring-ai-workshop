# Common Deployment Errors

| Error | Description | Resolution |
|--------|--------------|------------|
| ImagePullBackOff | Registry authentication issue | Verify Docker credentials |
| CrashLoopBackOff | Container repeatedly failing | Check logs via `kubectl logs` |
| Port already in use | Service conflicts | Change `server.port` in application.yml |
