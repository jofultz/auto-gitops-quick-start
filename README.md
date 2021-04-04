# Auto GitOps Quick Start

`AutoGitOps` is a dotnet tool that provides a basic templating engine for use with GitOps (Flux). This quick start walks through using `AutoGitOps`.

## Key Concepts

- Each `Kubernetes Cluster` has a directory in the `deploy` folder. In a real environment, this would be your centralized Flux repository. The repository we use is [here](https://github.com/retaildevcrews/gitops)
  - Each `cluster` contains a `config.json` file. This file allows you to define simple values that are applied to the `application templates`. In our example, we define `zone`, `region` and `template`.
  - The `template` value defines which `application template` to use for publishing. In our case, we use [AKDC](https://github.com/microsoft/kubernetes-developer-cluster-kubeadm) for our developer clusters.
    - default value is `autogitops.yaml`
- Each `Application` defines the application specific values and template(s)
  - `autogitops.json` defines the `values` used in substitution
  - `autogitops.yaml` (or *.yaml) is the template used for deployment
    - because our deployment is `simple` we chose not to use `Helm charts`
    - `Helm charts` are fully supported

## Install `AutoGitOps`

> `AutoGitOps` is packaged as a `dotnet tool` and a `Docker container`

### Install `AutoGitOps` as a dotnet global tool

> Requires dotnet core 5.0

```bash

# clone this repo
git clone https://github.com/bartr/auto-gitops-quick-start agoqs

# start in the root of the repo
cd agoqs

# uninstall (if necessary)
dotnet tool uninstall -g autogitops

# install latest beta version from this repo
dotnet tool install -g --add-source nupkg --version 0.0.2-beta-5 autogitops

```

### Run `AutoGitOps` on local data (dotnet tool)

```bash

# display usage
ago --help

# dry run with local files
ago --local-dev --dry-run

# merge local files
ago --local-dev

# see changes
git status

# undo any changes for future tests

```

### Run `AutoGitOps` on a GitHub repo

> WARNING: This will commit to your repo

- the default template directory is `autogitops`
  - use `--template-dir` to change

- `AutoGitOps` clones the repo to `../run_autogitops`
  - you have to have permission to the `..` directory
  - you have to remove `../run_autogitops` manually
  - `AutoGitOps` will fail if `../run_autogitops` exists
- If you don't have permission to commit to the repo, `AutoGitOps` will show as failed
  - the changes will be locally in `../run_autogitops`

```bash

# change the user, repo and PAT parameters
ago --ago-user bartr --ago-repo /bartr/gitops --ago-pat yourPAT

# you can set env vars and don't have to pass the values each time
# On Windows, use set
export AGO_USER bartr
export AGO_REPO /bartr/gitops
export AGO_PAT yourPAT

```

## TODO - update Docker instructions

### Install `AutoGitOps` Docker image

> WARNING: This is currently under development and likely doesn't work

```bash

docker pull ghcr.io/bartr/autogitops:beta

```

### Run `AutoGitOps` on local data (Docker)

- Mount your local template directory using `-v`
  - example: `-v $PWD/autogitops:/autogitops`

```bash

# remove (if necessary)
docker rm autogitops

docker run -it --name autogitops \
-v $PWD/autogitops:/autogitops \
-v $PWD/deploy:/deploy \
ghcr.io/bartr/autogitops:beta \
--local-dev

```

### Run `AutoGitOps` on a GitHub Repo (Docker)

- Mount your template directory using `-v`
  - example: `-v $PWD/autogitops:/autogitops`

```bash

# remove (if necessary)
docker rm autogitops

docker run -it --name autogitops \
-v $PWD/autogitops:/autogitops \
ghcr.io/bartr/autogitops:beta \
--ago-repo /retaildevcrews/gitops \
--ago-user yourGitHubName \
--ago-pat yourGitHubPAT

```
