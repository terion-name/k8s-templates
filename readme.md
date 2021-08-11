# Examples

## Minimal app with deployment, service and https-ingress
Other resources can be added in your kustomization

```yaml
# 19 lines without comments
# base imports
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - https://github.com/terion-name/k8s-templates.git/packs/default_https/
replacements:
  - path: https://raw.githubusercontent.com/terion-name/k8s-templates/master/helpers/name-replacement.yml
  - path: https://raw.githubusercontent.com/terion-name/k8s-templates/master/helpers/host-replacement.yml
# now app-specific customizations go
# set your namespace:
namespace: my-ns
# set app and host labels (and own if needed)
# app label will be used as application name
# host will be injected to ingress
# to add more additional hosts use patches
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
  - name: app
    envs:
      - app.env # FOO-bar
# add more stuff if needed
# initContainers can be also added with patch
```
<details>
 <summary>Will output the foollowing:</summary>

```yml
apiVersion: v1
data:
  FOO: bar
kind: ConfigMap
metadata:
  labels:
    app: my-awesome-app
    host: my-awesome-app.com
  name: app-59m54fbgh2
  namespace: my-ns
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: my-awesome-app
    host: my-awesome-app.com
  name: my-awesome-app
  namespace: my-ns
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 3000
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
  name: my-awesome-app
  namespace: my-ns
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
        - env:
            - name: PORT
              value: "3000"
          image: path/to/my/image:latest
          imagePullPolicy: Always
          name: my-awesome-app
          ports:
            - containerPort: 3000
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
  name: my-awesome-app
  namespace: my-ns
spec:
  rules:
    - host: my-awesome-app.com
      http:
        paths:
          - backend:
              serviceName: my-awesome-app
              servicePort: 3000
            path: /
  tls:
    - hosts:
        - my-awesome-app.com
      secretName: host-tls
```
</details>
