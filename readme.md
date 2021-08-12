#WIP!

## Examples

### Minimal app with deployment, service and https-ingress
Other resources can be added in your kustomization

```yaml
# 20 lines without comments
# base imports
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - https://github.com/terion-name/k8s-templates.git/packs/default_https/
replacements:
  - path: https://raw.githubusercontent.com/terion-name/k8s-templates/master/helpers/host-replacement.yml
# now app-specific customizations go
# set your namespace:
namespace: my-namespace
# set prefix, app and host labels (and own if needed)
# prefix will define all resources names
# host will be injected to ingress
# to add more additional hosts use patches
namePrefix: my-awesome-app-
commonLabels:
  app: my-awesome-app
  host: my-awesome-app.com
# set container image:
images:
  - name: app-image
    newTag: latest
    newName: path/to/my/image
# add env (or secrets with secretGenerator) if required:
configMapGenerator:
  - name: my-awesome-app-config # must be namePrefix + "config"
    behavior: merge # or replace. default config contains only PORT=5000
    envs:
      - app.env # FOO=bar, or inline values
# add more stuff if needed
# initContainers can be also added with patch
```
<details>
 <summary>Will output the foollowing:</summary>

```yml
apiVersion: v1
data:
  FOO: bar
  PORT: "5000"
kind: ConfigMap
metadata:
  labels:
    app: my-awesome-app
    host: my-awesome-app.com
  name: my-awesome-app-config
  namespace: my-namespace
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: my-awesome-app
    host: my-awesome-app.com
  name: my-awesome-app-service
  namespace: my-namespace
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort:
        valueFrom:
          configMapKeyRef:
            key: PORT
            name: config
  selector:
    app: my-awesome-app
    host: my-awesome-app.com
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-awesome-app
    host: my-awesome-app.com
  name: my-awesome-app-deployment
  namespace: my-namespace
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-awesome-app
      host: my-awesome-app.com
  template:
    metadata:
      labels:
        app: my-awesome-app
        host: my-awesome-app.com
    spec:
      containers:
        - envFrom:
            - configMapRef:
                name: my-awesome-app-config
          image: path/to/my/image:latest
          imagePullPolicy: Always
          name: main-container
          ports:
            - containerPort:
                valueFrom:
                  configMapKeyRef:
                    key: PORT
                    name: config
          resources:
            limits:
              memory: 1024Mi
            requests:
              memory: 1024Mi
      imagePullSecrets:
        - name: regcred
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
    certmanager.k8s.io/cluster-issuer: letsencrypt
    kubernetes.io/ingress.class: nginx
    kubernetes.io/tls-acme: "true"
  labels:
    app: my-awesome-app
    host: my-awesome-app.com
  name: my-awesome-app-ingress
  namespace: my-namespace
spec:
  rules:
    - host: my-awesome-app.com
      http:
        paths:
          - backend:
              serviceName: my-awesome-app-service
              servicePort: 80
            path: /
  tls:
    - hosts:
        - my-awesome-app.com
      secretName: my-awesome-app.com
```
</details>
