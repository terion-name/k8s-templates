# Examples

## Minimal app with deployment, service and https-ingress
Other resources can be added in your kustomization

```yaml
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
# set container image:
images:
  - name: app-image
    newTag: latest
    newName: path/to/my/image
# add env (or secrets with secretGenerator) if required:
configMapGenerator:
  - name: app
    envs:
      - app.env # FOO=bar
patches:
  # rename the deployment, all other resources will be renamed and matched after it:
  - target:
      kind: Deployment
      name: app
    patch: |-
      - op: replace
        path: /metadata/name
        value: my-resource
  # add target host, tls host will be replaced after it:
  - target:
      kind: Ingress
      name: app
    patch: |-
      - op: replace
        path: /spec/rules/0/host
        value: my-resource.com
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
  name: app-59m54fbgh2
  namespace: my-ns
---
apiVersion: v1
kind: Service
metadata:
  name: my-resource
  namespace: my-ns
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 3000
  selector:
    app: my-resource
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: my-resource
  name: my-resource
  namespace: my-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-resource
  template:
    metadata:
      labels:
        app: my-resource
    spec:
      containers:
      - env:
        - name: PORT
          value: "3000"
        image: path/to/my/image:latest
        imagePullPolicy: Always
        name: my-resource
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
  name: my-resource
  namespace: my-ns
spec:
  rules:
  - host: my-resource.com
    http:
      paths:
      - backend:
          serviceName: my-resource
          servicePort: 3000
        path: /
  tls:
  - hosts:
    - my-resource.com
    secretName: host-tls
```
</details>
