# Skaffold V2 - Kpt v1.0.0-beta integration example

This example explains out of place hydration when rendering and applying resources
to your cluster using kpt v1 deployer on skaffold v2 branch.

This example is skaffold-ified example of [kpt-nginx-example](../kpt-nginx-example). 
Since skaffold needs to build, I have added a go helloworld example in the built artifact.
The `KptFile` in [kpt-nginx-example](../kpt-nginx-example/Kptfile) has transform [configuration](../kpt-nginx-example/Kptfile#L24)
which is same as `transform` config in [skaffold.yaml](./skaffold.yaml#L10)


1) To install skaffold V2 (mac only), please reach out to tejal29@
2) Run `skaffold dev`
```shell
skaffold-kpt-example git:(main) skaffold dev -d gcr.io/tejal-gke1
Listing files to watch...
 - skaffold-example
Generating tags...
 - skaffold-example -> gcr.io/tejal-gke1/skaffold-example:00200b9
Checking cache...
 - skaffold-example: Found Remotely
Starting render...
Package ".kpt-pipeline": 
[RUNNING] "gcr.io/kpt-fn/set-labels:v0.1"
[PASS] "gcr.io/kpt-fn/set-labels:v0.1" in 1.5s

Successfully executed 1 function(s) in 1 package(s).
Tags used in deployment:
 - skaffold-example -> gcr.io/tejal-gke1/skaffold-example:00200b9@sha256:4830d819b6f1ced78b64816608c3f1d9c5f1a0cceac51a8f6a2189fc97815f41
Starting deploy...
initializing Kptfile inventory info (namespace: default)...success
service/my-nginx-svc apply failed: can't apply the resource since its annotation config.k8s.io/owning-inventory is a different inventory object
deployment.apps/skaffold-example apply failed: error when creating "manifests.yaml": Deployment.apps "skaffold-example" is invalid: [metadata.labels: Invalid value: " dev": a valid label must be an empty string or consist of alphanumeric characters, '-', '_' or '.', and must start and end with an alphanumeric character (e.g. 'MyValue',  or 'my_value',  or '12345', regex used for validation is '(([A-Za-z0-9][-A-Za-z0-9_.]*)?[A-Za-z0-9])?'), spec.selector.matchLabels: Invalid value: " dev": a valid label must be an empty string or consist of alphanumeric characters, '-', '_' or '.', and must start and end with an alphanumeric character (e.g. 'MyValue',  or 'my_value',  or '12345', regex used for validation is '(([A-Za-z0-9][-A-Za-z0-9_.]*)?[A-Za-z0-9])?'), spec.selector: Invalid value: v1.LabelSelector{MatchLabels:map[string]string{"app":"nginx", "env":" dev"}, MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: invalid label selector]
2 resource(s) applied. 0 created, 0 unchanged, 0 configured, 2 failed
E0404 20:09:17.859125   63733 task.go:248] Empty object UID from InventoryManager: default_my-nginx-svc__Service
E0404 20:09:17.864779   63733 task.go:248] Empty object UID from InventoryManager: default_skaffold-example_apps_Deployment
service/my-nginx-svc reconcile skipped
deployment.apps/skaffold-example reconcile skipped
0 resource(s) reconciled, 2 skipped, 0 failed to reconcile, 0 timed out
2 resources failed 
Cleaning up...
```

3) If you run `git status`, you will see the `deployment.yaml` 
and `svc.yaml` are unchanged. A new `.kpt-pipeline` directory exists.

Note: This temporary directory should be cleaned up at the end of the run. <TODO: tejal29@ to open a skaffold issues>

```shell
git status
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	.kpt-pipeline/
  nothing added to commit but untracked files present (use "git add" to track)	
```

You can take a look at the manifests.yaml in the `.kpt-pipeline`

```shell
skaffold-kpt-example git:(main) ✗ ls .kpt-pipeline 
Kptfile        manifests.yaml
```
<details>
  <summary> Manifests.yaml contains the hydrated manifests with label `env: dev`</summary>

```shell 
skaffold-kpt-example git:(main) ✗ cat .kpt-pipeline/manifests.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    skaffold.dev/run-id: 6ceeed78-8a8e-42f1-be2b-8ddb74416fcb
    env: ' dev'
  name: skaffold-example
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
      env: ' dev'
  template:
    metadata:
      labels:
        app: nginx
        skaffold.dev/run-id: 6ceeed78-8a8e-42f1-be2b-8ddb74416fcb
        env: ' dev'
    spec:
      containers:
      - image: gcr.io/tejal-gke1/skaffold-example:00200b9@sha256:4830d819b6f1ced78b64816608c3f1d9c5f1a0cceac51a8f6a2189fc97815f41
        name: nginx
        ports:
        - containerPort: 80
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
    skaffold.dev/run-id: 6ceeed78-8a8e-42f1-be2b-8ddb74416fcb
    env: ' dev'
  name: my-nginx-svc
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
    env: ' dev'
  type: LoadBalancer
➜  skaffold-kpt-example git:(main) ✗ 
```
