## About Openshift Route


## How to integrate Openshift Route with Argo Rollouts


## Requirements
- OpenShift cluster with support for routes installed
- Argo Rollouts installed (see [install guide](../../installation.md))

!!! tip
    Make sure you are either switched to the correct namespace or you specify the namespace to be used in the following examples. It is assumed you are working on the `argo-rollouts` namespace from the installation guide.

NOTES:

**_1. The file as follows (and the codes in it) just for illustrative purposes only, please do not use directly!!!_**

**_2. The argo-rollouts >=  _**

Steps:

1. Run the `yaml/rbac.yaml` to add the role for operate on the `Route`.
2. Build this plugin.
3. Put the plugin somewhere & mount on to the `argo-rollouts`container (please refer to the example YAML below to modify the deployment):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: argo-rollouts
  namespace: argo-rollouts
spec:
  template:
    spec:
      ...
      volumes:
        ...
         - name: openshift-plugin
           hostPath:
             path: /CHANGE-ME/rollouts-plugin-trafficrouter-openshift
             type: ''
      containers:
        - name: argo-rollouts
        ...
          volumeMounts:
             - name: openshift-plugin
               mountPath: /CHANGE-ME/rollouts-plugin-trafficrouter-openshift

```

4. Create a ConfigMap to let `argo-rollouts` know the plugin's location:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argo-rollouts-config
  namespace: argo-rollouts
data:
  trafficRouterPlugins: |-
    - name: "argoproj-labs/openshift"
      location: "file://CHANGE-ME/rollouts-trafficrouter-openshift-plugin/openshift-plugin"
binaryData: {}
```

5. Create the `CR/Rollout` and put it into the operated services` namespace:

```yaml

apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: rollouts-demo
  namespace: rollouts-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/instance: rollouts-demo
  strategy:
    canary:
      canaryService: canaryService
      stableService: stableService
      steps:
        - setWeight: 30
        - pause:
            duration: 10
      trafficRouting:
        plugins:
          argoproj-labs/openshift:
            routes:
              - rollouts-demo
            namespace: rollouts-demo
  workloadRef:
    apiVersion: apps/v1
    kind: Deployment
    name: canary
```

6. Enjoy It.

## Contributing

Thanks for taking the time to join our community and start contributing!

- Please familiarize yourself with the [Code of Conduct](/CODE_OF_CONDUCT.md) before contributing.
- Check out the [open issues](https://github.com/argoproj-labs/rollouts-plugin-trafficrouter-openshift/issues).