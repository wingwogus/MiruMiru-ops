# Cluster Bootstrap

Run these commands after Argo CD is installed in the cluster.

## 1. Namespace

```bash
kubectl create namespace mirumiru
```

## 2. App secrets and config

Apply the MiruMiru app secrets and config into the `mirumiru` namespace.

Expected names:
- `mirumiru-api-secret`
- `mirumiru-api-config`

The app chart references those names directly.

## 3. Image pull secret for the app

```bash
kubectl create secret docker-registry regcred \
  -n mirumiru \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username='<DOCKERHUB_USERNAME>' \
  --docker-password='<DOCKERHUB_PASSWORD>'
```

## 4. Git credentials for Image Updater write-back

```bash
kubectl -n argocd create secret generic git-creds \
  --from-literal=username='<GITHUB_USERNAME>' \
  --from-literal=password='<GITHUB_TOKEN_WITH_REPO_WRITE>'
```

## 5. Docker Hub credentials for Image Updater

```bash
kubectl -n argocd create secret docker-registry dockerhub-secret \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username='<DOCKERHUB_USERNAME>' \
  --docker-password='<DOCKERHUB_PASSWORD>'
```

## 6. Registry configuration for Image Updater

```bash
cat <<'EOF' | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-image-updater-config
  namespace: argocd
data:
  registries.conf: |
    registries:
      - name: Docker Hub
        prefix: docker.io
        api_url: https://registry-1.docker.io
        ping: yes
        credentials: pullsecret:argocd/dockerhub-secret
EOF
```

Restart Image Updater after changing the registry config:

```bash
kubectl rollout restart deployment/argocd-image-updater -n argocd
```

## 7. Apply GitOps resources

```bash
kubectl apply -f argocd/mirumiru-prod.yaml
kubectl apply -f image-updater/mirumiru-updater.yaml
```
