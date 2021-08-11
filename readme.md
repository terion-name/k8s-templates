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
      - app.env
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
