
...Under construction...

# Intoduction

The Kubernetes Dashboard is a container that can be install in your Kubernetes cluster and provide visual feedback on the cluser and active jobs, rather than through multiple command line arguments.

To install the kubernetes cluster
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

To enable access to the dashboard (must be running; i.e. must restart after closing the terminal, or upon restart)
```
kubectl proxy
```

To access the dashboard in the browser, go to
```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

You may be required to enter a token, which can be created for an existing service account for the kubernetes dashboard
To see the accounts use the following. Typically there is a 'default' and 'kubernetes-dashboard' account
```
kubectl -n kubernetes-dashboard get serviceaccounts
```

Create a token for the account, specifying the service account as the last argument in the following:
```
kubectl -n kubernetes-dashboard create token default
```

A token will be output such as the following, which can be directly entered as the token when accessing the dashboard in the browser
```
eyJhbGciOiJSUzI1NiIsImtpZCI6IkpsY1N4Mm9tMWxtTlRWWXhGTFNyd0JKNmZsb3lCS3F4aHE3YkVKQmlRM2sifQ
...
...
...
...
_vi1010UDT8RGEfB0Xn9aCC4ZVUIlxIsY5Il6iTNBL7LPkKZQ8Ved3WEBY-Iw5mv461Q
```

# Additional Resources

For more information see
- Creating tokens https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md
- Kubernetes documentation on the Dashboard https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/