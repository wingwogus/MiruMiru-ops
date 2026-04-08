# MiruMiru Ops Repo

This repository is the GitOps source of truth for MiruMiru deployment state.

It holds:
- the Helm chart used by Argo CD
- Argo CD `Application` manifests
- Argo CD Image Updater configuration
- bootstrap references for cluster-side secrets

It intentionally does **not** hold application source code or CI build logic. The app repo builds and pushes container images. This repo declares which image tag the cluster should run.

## Layout

- `apps/mirumiru/`: Helm chart for the backend deployment
- `argocd/mirumiru-prod.yaml`: Argo CD Application for the production cluster
- `image-updater/mirumiru-updater.yaml`: Argo CD Image Updater configuration
- `bootstrap/README.md`: one-time cluster bootstrap commands and secrets

## Bootstrap flow

1. Create this repository remotely, for example `wingwogus/MiruMiru-ops`.
2. Push this local repo to that remote.
3. Update `repoURL` in `argocd/mirumiru-prod.yaml` if the final repo URL differs.
4. Apply `argocd/mirumiru-prod.yaml` into the cluster.
5. Create the secrets described in `bootstrap/README.md`.
6. Apply the Image Updater registry config and `image-updater/mirumiru-updater.yaml`.

## Notes

- The chart defaults to `latest` as a bootstrap tag, but Image Updater is configured to move the deployment to commit-SHA tags.
- The app repo CI should only build and push images. It should not mutate deployment manifests.
