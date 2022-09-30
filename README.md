
# Nginx ingress across namespaces
If you have multiple apps across different namespaces in a single kubernetes cluster, and you
want to use a single ingress with wildcard cert for example, use this example.


## Important Notes

### The error when describing ingress
Note: `kubectl describe ing <ingress name>` will show the following error:

```
  Host              Path  Backends
  ----              ----  --------
  app1.example.com
                    /   app1:80 (<error: endpoints "app1" not found>)
  app2.example.com
                    /   app2:80 (<error: endpoints "app2" not found>)

```

You can ignore this. Read [THIS](https://stackoverflow.com/a/67180704) for more context

### Nginx vs GCE ingress controller

This does not work with GCE ingress controller because GCE needs healthchecks on the service.
Not sure if there is a workaround, I did not check it. Feel free to investigate further if you 
want to use gce ingress controller.



## The Setup

- `ingress` : This directory contains the common ingress and servces with `ExternalName` type (read below)
- `app1`: Everything related to the app1 (deployment, service and namespace)
- `app2`: Everything related to the app2 (deployment, service and namespace)


## How does it work?

By default, any ingress works only if the backends are in the same namespace. This is by design.
to circumvent this, we use `type: ExternalName` Kubernetes service. 

Example:

```
kind: Service
apiVersion: v1
metadata:
  name: app1
  namespace: default
spec:
  type: ExternalName
  externalName: app1-svc.app1.svc.cluster.local
  ports:
  - port: 80
```

When we do this, we are creating a service `app1` in the `default` namespace which does not have a selector,
instead it uses the DNS name to reach the actual service under the `app1` namespace.


