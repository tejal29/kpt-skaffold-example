# Kpt v1.0.0-beta example

This example explains kpt v1.0.0-beta's in place hydration when rendering and applying resources
to your cluster.

1) Install Kpt v1.0.0-beta https://kpt.dev/installation/
2) Run
```shell
kpt fn render
```
3) Run `git diff` and you will kpt applied the mutations mentioned in the pipeline
directly to the k8 resource yaml

```shell
diff --git a/kpt-nginx-example/Kptfile b/kpt-nginx-example/Kptfile
index f6de9b6..c6c4793 100644
--- a/kpt-nginx-example/Kptfile
+++ b/kpt-nginx-example/Kptfile
@@ -2,6 +2,8 @@ apiVersion: kpt.dev/v1
 kind: Kptfile
 metadata:
   name: nginx
+  labels:
+    env: dev
 upstream:
   type: git
   git:
diff --git a/kpt-nginx-example/deployment.yaml b/kpt-nginx-example/deployment.yaml
index 478a6b7..eb44d1d 100644
--- a/kpt-nginx-example/deployment.yaml
+++ b/kpt-nginx-example/deployment.yaml
@@ -11,24 +11,27 @@
 # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 # See the License for the specific language governing permissions and
 # limitations under the License.
-
 apiVersion: apps/v1
 kind: Deployment
 metadata: # kpt-merge: /my-nginx
   name: my-nginx
+  labels:
+    env: dev
 spec:
   replicas: 4
   selector:
     matchLabels:
       app: nginx
+      env: dev
   template:
     metadata:
       labels:
         app: nginx
+        env: dev
     spec:
diff --git a/kpt-nginx-example/svc.yaml b/kpt-nginx-example/svc.yaml
index 25fb693..84c9733 100644
--- a/kpt-nginx-example/svc.yaml
+++ b/kpt-nginx-example/svc.yaml
@@ -11,17 +11,18 @@
 # WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 # See the License for the specific language governing permissions and
 # limitations under the License.
 apiVersion: v1
 kind: Service
 metadata: # kpt-merge: /my-nginx-svc
   name: my-nginx-svc
   labels:
     app: nginx
+    env: dev
 spec:
   type: LoadBalancer
   selector:
     app: nginx
+    env: dev
   ports:
     - protocol: TCP
       port: 80
```

4) Now lets, try to apply this resources on our cluster.

Before, applying lets first commit these changes. 

```shell
git add . 
git commit -am "render commit"

```

Now, initialize apply

```shell
kpt live init  
initializing Kptfile inventory info (namespace: default)...success
```

Lets finally apply, 

```shell
kpt live apply
```

If you see, kpt v1 made in place changes to the `Kptfile`

```shell
diff --git a/kpt-nginx-example/Kptfile b/kpt-nginx-example/Kptfile
index c6c4793..c43020a 100644
--- a/kpt-nginx-example/Kptfile
+++ b/kpt-nginx-example/Kptfile
@@ -27,3 +27,7 @@ pipeline:
     - image: gcr.io/kpt-fn/set-labels:v0.1
       configMap:
         env: dev
+inventory:
+  namespace: default
+  name: inventory-31552507
+  inventoryID: b7cf54f2c3b853091772048dd0e7e9d84c6cf5a0-1649100082385222000
(END)
```
