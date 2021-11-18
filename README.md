# ArgoFlow

> :warning: This repo is already configured for a specific usage, follow this guide on a new Argoflow fork.

## First Install

### Setup Repository

Fork [argoflow](https://github.com/argoflow/argoflow) repository on GitHub

Copy example `setup.conf` file `cp examples/setup.conf setup.conf`
Edit `setup.conf` file to your needs. To make argocd always watch the last commit, set `<<__git_repo.target_revision__>>=HEAD`.

### Install Sealed Secrets

Install sealed-secrets controller which allows for having secrets in (public) repositories securely.

```bash
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install -n kube-system sealed-secrets-controller sealed-secrets/sealed-secrets
```

### Apply configuration

Execute `setup_repo.sh setup.conf` to setup the repository.

Commit and push this initial configuration.

```bash
git add .
git commit -m "conf"
git push
```

### Install Argo-CD

Depending on wether your repository is public or private, install one of the two deployment.

```bash
kustomize build distribution/argocd/base | kubectl apply -f -
# OR (private repo)
kustomize build distribution/argocd/overlays/private-repo | kubectl apply -f -
```

If your private repo credentials are not taken into account, add it mannually by following this [guide](https://argo-cd.readthedocs.io/en/release-1.8/user-guide/private-repositories/). I advice using SSH Private Key Credential method.

### Edit application configuration

Open `distribution/kustomize.yaml` file and chose which app you want to installed.
You can also edit kustomize files for each app to match your needs.

Don't forget to commit and push your changes.

### Deploy all apps

Run `kubectl apply -f distribution/kubeflow.yaml` to deploy all apps.

## Kubeflow configuration

### Edit Kubeflow notebook images

To changes which images are available in Kubeflow notebook, edit file `distribution/kubeflow/notebooks/jupyter-web-app/spawner_ui_config.yaml`.

- Change `spawnerFormDefaults.image.values` with your default image.
- Change `spawnerFormDefaults.image.options` with your list of available images. You also have to add the default image to the list.

> :warning: Be careful, image used by Kubeflow must respect some constraints described [here](https://www.kubeflow.org/docs/components/notebooks/custom-notebook/).

## DaskHub

A Daskhub application has been added. It's uses daskhub chart has chart dependency to allow simpler values override. Just edit `distribution/daskhub/values.yaml` file.

There is also an ArgoCD application in `distribution/argocd-applications/daskhub.yaml` file. The application can be enabled in `distribution/kustomize.yaml` by uncommenting the corresponding line.
