# Auto GitOps Quick Start

`AutoGitOps` is a tool that provides a basic templating engine for use with [Flux](https://fluxcd.io/). This quick start walks through using `AutoGitOps`.

## Key Concepts

- `AutoGitOps` separates `Platform` from `Apps`
  - Platform teams are responsible for the GitOps Repo
  - Application teams are responsible for `values` and `templates`
  - This separation empowers application teams while providing centralized cluster management
- Each `Kubernetes Cluster` has a directory in the `deploy` folder. In a real environment, this would be your centralized Flux repository maintained by the `platform team`.
  - A sample repository is [here](https://github.com/bartr/gitops)
    - Each `cluster` contains a `config.json` file. This file allows you to define simple values that are applied to the `application templates`. In our example, we define `zone`, `region` and `template`.
    - The `template` value defines which `application template` to use for publishing. In our case, we use [AKDC](https://github.com/microsoft/kubernetes-developer-cluster-kubeadm) for our developer clusters.
      - default value is `autogitops.yaml`
- Each `Application Team` defines the application specific values and template(s)
  - `autogitops.json` defines the `values` used in substitution
    - Required values
      - name
      - namespace
      - targets
        - targets map to the directories in the `deploy` directory of the `GitOps Repo`
  - `autogitops.yaml` (or *.yaml) is the template used for deployment
    - because our deployment is `simple` we chose not to use `Helm charts`
    - the templating capability of `AutoGitOps` often replaces the need for Helm charts
    - `Helm charts` are fully supported

## Install `AutoGitOps`

> `AutoGitOps` is packaged as a `dotnet tool` and a `Docker image`

```bash

# clone this repo
git clone https://github.com/bartr/auto-gitops-quick-start agoqs

# start in the root of the repo
cd agoqs

```

### Install `AutoGitOps` as a dotnet global tool

> Requires dotnet core >= 3.1

```bash

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

### Run `AutoGitOps` on a GitOps repo

> WARNING: This will commit to your repo (if you have permission)

- the default template directory is `autogitops`
  - use `--template-dir` to change

- `AutoGitOps` clones the repo to `../run_autogitops`
  - you have to have permission to the `..` directory
- If you don't have permission to commit to the repo, `AutoGitOps` will show as failed
  - the changes will be in `../run_autogitops`

```bash

# change the user, repo and PAT parameters
ago --ago-user bartr --ago-repo /bartr/gitops --ago-pat yourPAT

# you can set env vars and don't have to pass the values each time
export AGO_USER bartr
export AGO_REPO /bartr/gitops
export AGO_PAT yourPAT

```

### Install `AutoGitOps` Docker image

```bash

docker pull ghcr.io/bartr/autogitops

```

### Volume Mounts

- Mount your local template directory using `-v`
  - example: `-v $PWD/autogitops:/app/autogitops`
- Mount your local deploy directory using `-v`
  - example: `-v $PWD/autogitops:/app/deploy`

### Run `AutoGitOps` interactively (Docker)

```bash

docker run -it --rm \
-v $PWD/autogitops:/app/autogitops \
-v $PWD/deploy:/app/deploy \
--entrypoint /bin/bash \
ghcr.io/bartr/autogitops

# from the docker container
./ago --local-dev --dry-run

# apply the template locally
./ago --local-dev

# apply the template to a GitOps repo
./ago -u yourUser -r /bartr/gitops -p yourPAT

# exit the docker container
exit

# see the updates
git status

# undo any updates

```

### Run `AutoGitOps` on local data (Docker)

```bash

docker run -it --rm \
-v $PWD/autogitops:/app/autogitops \
-v $PWD/deploy:/app/deploy \
ghcr.io/bartr/autogitops \
--local-dev

# see the updates
git status

# undo any updates

```

### Run `AutoGitOps` on a GitHub Repo (Docker)

- Mount your template directory using `-v`
  - example: `-v $PWD/autogitops:/app/autogitops`

```bash

docker run -it --rm \
-v $PWD/autogitops:/app/autogitops \
ghcr.io/bartr/autogitops \
--ago-repo $AGO_REPO \
--ago-pat $AGO_PAT \
--ago-user $AGO_USER

```
