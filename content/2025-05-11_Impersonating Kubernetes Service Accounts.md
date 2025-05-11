When developing applications with Kubernetes, it's common to use service accounts. You may need to impersonate a service account to test application.

A simple way to do this would be to generate a token for the service account:

```bash
kubectl create token <service-account-name> --duration=12h
```

then insert the token into your `~/.kube/config` file:

```yaml
users:
  - name: <service-account-name>
    user:
      token: <token>
```

However, you have to repeat this process every time the token expires.

To automate this process, you can configure the user in your `~/.kube/config` file like this:

```yaml
users:
  - name: <service-account-name>
    user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      args:
        - -c
        - echo '{"apiVersion":"client.authentication.k8s.io/v1beta1","kind":"ExecCredential","status":{"token":"'$(kubectl create token <service-account-name> --duration=12h)'"}}'
      command: bash
```
