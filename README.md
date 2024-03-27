
 ###### last updated: 2024-03-25

 ### Overview

 The repository provides necessary stuff to build AlmaLinux **Container Images** (Docker Images), test them, push into deferent registries [Docker.io](https://hub.docker.com), [Quay.io](https://quay.io), [GitHub Packages](https://github.com/features/packages) (the *Client Library*) and publish to [Docker *Official Library*](https://hub.docker.com/u/library).

The Container Images can be used with all [OCI complaint](https://opencontainers.org/) container runtime environments like Docker, Podman and Kubernetes and serve as drop-in replacements for centos images as it reaches [End of Life](https://centos.org/centos-linux-eol/).

 ### Requirements/Prerequisites

Personal, Organization or Enterprise account on GitHub is the only requirement. Please read more about [accounts on GitHub](https://docs.github.com/en/enterprise-server@3.10/get-started/learning-about-github/types-of-github-accounts).

 ### The idea

 The project utilizes [GitHub Actions](https://github.com/features/actions) to provide public, transparent and fast workflows which are easy to understand, use and modify.

Workflows on Github Actions:
- build and push container images into the *Client Library*
- use some of these images to request Docker to create ones for the *Official Library*.

The AlmaLinux ***Client Library*** includes the following registries/organizations:
- [Docker.io/almalinux](https://hub.docker.com/u/almalinux) (Sponsored by OSS)
- [Quay.io/almalinuxorg](https://quay.io/organization/almalinuxorg)
- [Ghcr.io/AlmaLinux](https://github.com/orgs/AlmaLinux/packages)

The AlmaLinux [***Official Library***](https://hub.docker.com/_/almalinux) is maintained by Docker.

 ### Workflows

So there are two workflows to achieve the idea:
 1. **Build, test and push to the Client Library** all types of container images.

 2. **Publish images to the Docker Library** (`default` and `minimal` configurations types only).

> Read more about how these [**workflows work**](#workflows-jobs-and-steps)

 ### Containerfiles

Each image pushed to the *Client Library* is build from its corresponded [Containerfile](https://github.com/AlmaLinux/container-images/tree/main/Containerfiles). There are uniq files for each AlmaLinux release and configuration types: `base`, `default`, `init`, `micro`, `minimal`.
These files match [Dockerfile](https://docs.docker.com/reference/dockerfile/) standard. There are commands and instructions to install AlmaLinux whole root filesystem in them.

Images for the Docker *Official Library* are build using other and different [Containerfiles](https://github.com/AlmaLinux/container-images/tree/docker-library/Containerfiles). They are also for each AlmaLinux release but only `default` and `minimal` types. These Containerfiles source corresponded images from the *Client Library* at [Quay.io/almalinuxorg](https://quay.io/organization/almalinuxorg)

## What Container Images are built

### AlmaLinux releases

Container images for AlmaLinux releases 8 and 9 are built. The release (Major version) should be chosen for the **Build, Test and Push** workflow. The Minor version is automatically set by the workflow into the latest, so images are built with AlmaLinux 8.9 or 9.3 (releases are as of 19 Mar 2024) installed.

**Publish Images** workflow pushes build request to the Docker for AlmaLinux both 8 and 9 releases.

### Image configuration types

AlmaLinux container images types match [Red Hat Universal Base Image](https://catalog.redhat.com/software/base-images):
- `base`
- `default` (the image is also available via the Docker *Official Library*)
- `init`
- `micro`
- `minimal` (the image is also available via the Docker *Official Library*)

### Supported platforms/architectures

**Build, Test and Push** workflow build container images of the following platforms simultaneously with `docker buildx`. They result into the following machine hardware names (`uname -m`):

| docker platform | hardware name |
| --------------- | ------------- |
| linux/amd64     | x86_64        |
| linux/ppc64le   | ppc64le       |
| linux/s390x     | s390x         |
| linux/arm64     | aarch64       |

The [**containerd image store store**](https://docs.docker.com/storage/containerd/) for Docker Engine together with `buildx` are used to build and push multiple platforms at ones.

### Repositories

The following *repositories* are created on all registries ([Docker.io/almalinux](https://hub.docker.com/u/almalinux), [Quay.io/almalinuxorg](https://quay.io/organization/almalinuxorg), [Ghcr.io/AlmaLinux](https://github.com/orgs/AlmaLinux/packages)) and for supported images types:

- `/almalinux` - [Quay.io/almalinuxorg](https://quay.io/repository/almalinuxorg/almalinux) only. Is built from the `default` image.
- `/8-base`
- `/9-base`
- `/8-init`
- `/9-init`
- `/8-micro`
- `/9-micro`
- `/8-minimal`
- `/9-minimal`

They are the *Client Library*.

### Tags

The following tags are created under each *repository* (AlmaLinux 9.3 example as of 24 Nov 2023):

| tag                    | example       |
| ---------------------- | ------------- |
| latest                 | latest        |
| MAJOR                  | 9             |
| MAJOR.MINOR            | 9.3           |
| MAJOR.MINOR-DATE_STAMP | 9.3-20231124  |

The `/almalinux` *repository* will include the `latest` tag for AlmaLinux release 9.x only.

## *container-images* repository structure

### Directories structure

1. Branch 'main'
```sh
.
├── .github
│   └── workflows
│       ├── build-test-push.yml
│       └── publish-docker-library.yml
├── Containerfiles
│   ├── 8
│   │   ├── Containerfile.base
│   │   ├── Containerfile.default
│   │   ├── Containerfile.init
│   │   ├── Containerfile.micro
│   │   └── Containerfile.minimal
│   └── 9
│       ├── Containerfile.base
│       ├── Containerfile.default
│       ├── Containerfile.init
│       ├── Containerfile.micro
│       └── Containerfile.minimal
├── LICENSE
└── README.md
```

2. Branch 'docker-library'
```sh
.
├── Containerfiles
│   ├── 8
│   │   ├── Containerfile.default
│   │   └── Containerfile.minimal
│   └── 9
│       ├── Containerfile.default
│       └── Containerfile.minimal
└── docker-library-definition.tmpl
```

### Workflow **.yml* files

To **Build, Test and Push** images to the *Client Library* the [`.github/workflows/build-test-push.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/build-test-push.yml) workflow is used:
```yaml
name: Build, test and push to the Client Library

on:
  workflow_dispatch:
      inputs:
        production:
          description: |
            'Push to production registries'
            'not checked - to testing'
          required: true
          type: boolean
          default: true
...
```

To **Publish Images** to the Docker *Official Library* the
[`.github/workflows/publish-docker-library.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/publish-docker-library.yml) workflow is used:
```yaml
name: Publish images to the Docker Library

on:
  workflow_dispatch:
      inputs:
        pr:
          description: 'Publish to Docker Official Library'
          required: true
          type: boolean
          default: true
...
```

Both workflows are triggered manually using [**workflow_dispatch**](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch) event of GitHub Actions.

### Sourced *Containerfiles*

AlmaLinux release 9 and `minimal` example of [`Containerfiles/9/Containerfile.minimal`](https://github.com/AlmaLinux/container-images/blob/main/Containerfiles/9/Containerfile.minimal) file to build container image for the *Client Library*:
```Dockerfile
ARG SYSBASE=almalinux:9
FROM ${SYSBASE} as system-build

RUN mkdir /mnt/sys-root; \
    dnf install -y \
    --installroot /mnt/sys-root \
    --releasever 9 \
    --setopt install_weak_deps=false \
    --nodocs \
    almalinux-release \
    bash \

...

FROM scratch
COPY --from=system-build /mnt/sys-root/ /

CMD ["/bin/bash"]
```

AlmaLinux release 9 and `minimal` example of [`Containerfiles/9/Containerfile.minimal`](https://github.com/AlmaLinux/container-images/blob/docker-library/Containerfiles/9/Containerfile.minimal) file to build container image for the Docker *Official Library*:
```Dockerfile
# Tags: minimal, 9-minimal, 9.3-minimal, 9.3-minimal-20231124
FROM quay.io/almalinuxorg/9-minimal:9.3-20231124

CMD ["/bin/bash"]
```

### Template file for Docker *Library Definition*

The Docker *Official Library* uses [Definition File](https://github.com/docker-library/official-images/blob/master/library/almalinux) to request official images building. Changing the file triggers new image(s) building on Docker side. The [`docker-library-definition.tmpl`](https://github.com/yuravk/container-images/blob/docker-library/docker-library-definition.tmpl) template is used to generate the Definition file:
```yaml
Tags: {{ .tags }}
GitFetch: refs/heads/docker-library
GitCommit: {{ .commit_hash }}
File: Containerfiles/{{ .version_major }}/Containerfile.{{ .image_type }}
Architectures: amd64, arm64v8, ppc64le, s390x
```

# How to contribute/help and customize workflow(s)

## Fork GitHub repositories

Fork the following repositories:
- [**container-images**](https://github.com/AlmaLinux/container-images), you will ned both `main` and `container-library` branches.
- [**docker-library**](https://github.com/docker-library/official-images)

>Please note, you won't be able to create Pull Request with this repository (what **Publish Images** workflow does). That is allowed for AlmaLinux organization only.

Read more about GitHub [forks here](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo).

## Set Action's secrets

To set secrets, under your repository, click **Settings**, then expand **Secrets and variables** (under the **Security** section), and select **Actions** there. Please find more about [Github Secrets in Actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions).

The following *Repository secrets* are required. Set them with your personal data and ***only for registries you are using***.

For production *Client Library* please define secrets:
| Secret name            | Description          |
| ---------------------- | -------------------- |
| `DOCKERHUB_USERNAME`   | docker.io user       |
| `DOCKERHUB_TOKEN`      | docker.io token      |
| `QUAY_IO_USERNAME`     | quay.io user         |
| `QUAY_IO_CLI_PASSWORD` | quay.io CLI password |
| `GIT_HUB_USERNAME`     | GitHub user          |
| `GIT_HUB_TOKEN`        | GitHub token         |

The same secrets with `TEST_` prefix in secret names (like `TEST_DOCKERHUB_USERNAME`) should be set for corresponded registries if testing *Client Library* (testing mode).

On how to create tokens/CLI passwords please read:
- Manage **Quay.io** [Access Tokens](https://docs.quay.io/glossary/access-token.html)
- [Create and manage access tokens](https://docs.docker.com/security/for-developers/access-tokens/) on **Docker**
- [Managing your personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) on **GitHub**.

On **GitHub**, when you create new personal access token, please select:
- *write:packages* scope, to allow packages uploading to GitHub Package Registry;
- *admin:org* scope, to allow Pull Request creation to [docker-library](https://github.com/docker-library/official-images).

## Change registries list

You won't be able to use AlmaLinux accounts, so please create your own accounts (if you don't have) on need registries.

Then, having registries list and know your user names there, please change it in your [`.github/workflows/build-test-push.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/build-test-push.yml) workflow file (branch `master`):
```yaml
registries: 'docker.io/<user_name>, quay.io/<user_name>, ghcr.io/<user_name>'
```
Where, `<user_name>` - your user name on the specific registry. It can not be the same for all registries. Separate registries with the comma.

## Change platforms list

If you don't need to build images for all platforms, change theirs list in your [`.github/workflows/build-test-push.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/build-test-push.yml) workflow file (branch `master`):

```yaml
platforms: 'linux/amd64, linux/ppc64le, linux/s390x, linux/arm64'
```
Separate platforms with the comma.

## Change image types list

To change type of images which are build, edit your [`.github/workflows/build-test-push.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/build-test-push.yml) workflow file (branch `master`) and make corresponded changes:

- Add/modify/delete input for specific type name of image:
```yaml
        type_<type_name>:
          description: '<type_name>'
          required: true
          type: boolean
```

- Add/modify/delete the image type in the matrix for the `build` job:
```yaml
image_types: ${{ fromJSON(format('["{0}"]', ( inputs.type_<type_name> && '<type_name>' ) )) }}
```
Where, `<type_name>` - name of your image type.

Default are: *base*, *default*, *init*, *micro*, *minimal*

## To bump AlmaLinux release (*Major* number)

If AlmaLinux new release is available (the Major version), edit your [`.github/workflows/build-test-push.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/build-test-push.yml) workflow file (branch `master`), and set:

- `inputs.version_major` like:
```yaml
        version_major:
          description: 'AlmaLinux major version'
          required: true
          default: '<version_latest>'
          type: choice
          options:
            - <version_latest>
            - 9
            - 8
```
 - `env.version_latest` like:
```yaml
version_latest: <version_latest>
```
Where, `<version_latest>` - AlmaLinux version major number, for example `10`.

## To bump AlmaLinux release (*Minor* number)

If AlmaLinux new release is available (the Minor version), edit your [`.github/workflows/build-test-push.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/build-test-push.yml) workflow file (branch `master`), step "DeployPrepare AlmaLinux Minor version number":

```sh
          case ${{ inputs.version_major }} in
            8)
              version_minor="<8_minor>" ;;
            9)
              version_minor="<9_minor>"  ;;
            10)
              version_minor="<10_minor>" ;;

```

Where, `<8_minor>`, `<9_minor>`, `<10_minor>` - AlmaLinux corresponded version minor numbers.

For example Minors are `10`, `4` or `1` for new **8.10**, **9.4** or **10.1** versions respectively.

## Restrictions

> The AlmaLinux organization only is allowed to create Pull Requests and so publish container images into the *Docker Official Library*.

> **Build, test and push to the Client Library** workflow will work only for your *Client Library*, but not for AlmaLinux owned.

# Workflows jobs and steps

## Build, test and push to the Client Library

Tree illustration of the workflow Jobs and Steps for AlmaLinux 9 minimal image:
```
    Build, test and push to the Client Library
    │
    ├── Deploy 9 minimal images
    │   ├── Set up job
    │   ├── DeployPrepare AlmaLinux Minor version number
    │   ├── Prepare date stamp
    │   ├── Generate list of Docker images to use as base name for tags
    │   ├── Enable containerd image store on Docker Engine
    │   ├── Checkout _container-images, branch 'main'
    │   ├── Checkout _container-images, branch 'docker-library', path 'docker-library'
    │   ├── Set up QEMU
    │   ├── Set up Docker Buildx
    │   ├── Login to Docker.io
    │   ├── Login to Quay.io
    │   ├── Login to Ghcr.io
    │   ├── Generate tags and prepare metadata to build and push
    │   ├── Build images
    │   ├── Test images
    │   ├── Push images to Client Library
    │   ├── Change date stamp in Containerfile (default and minimal only)
    │   ├── Upload changed Containerfile (default and minimal only)
    │   ├── Post Push images to Client Library
    │   ├── Post Build images
    │   ├── Post Login to Ghcr.io
    │   ├── Post Set up Docker Buildx
    │   ├── Post Checkout _container-images, branch 'docker-library', path 'docker-library'
    │   ├── Post Checkout _container-images, branch 'main'
    │   └── Complete job
    │
    └── Collect and save changed Containerfile(s) used by Docker Official Library
        ├── Set up job
        ├── Checkout container-images, branch 'docker-library'
        ├── Download changed Containerfiles
        ├── [Debug] Print Containerfiles/9/Containerfile.
        ├── Commit and push Containerfiles/9/Containerfile.minimal changes
        ├── Post Checkout container-images, branch 'docker-library'
        └── Complete job
```

### Inputs

The workflows inputs are:
- `production` - boolean '*Push to production registries*' with the default value `true` (checked). Container images are pushed into the production *Client Library*: [Docker.io/almalinux](https://hub.docker.com/u/almalinux), [Quay.io/almalinuxorg](https://quay.io/organization/almalinuxorg) and [Ghcr.io/AlmaLinux](https://github.com/orgs/AlmaLinux/packages). Otherwise, images are pushed into the testing *Client Library*: [Quay.io/almalinuxautobot](https://quay.io/organization/almalinuxautobot)

- `version_major` - dropdown '*AlmaLinux major version*' with the default value `9`. That is major number of AlmaLinux version to build images for.

- Checklist of image types: *base*, *default*, *init*, *micro*, *minimal*. At least one should be checked.

### Job: Deploy *version_major* *image_types* images

Job is proceed for input `version_major` and iterates (using matrix) with selected `image_types`. Multiple jobs run simultaneously over each image type.

#### Step: DeployPrepare AlmaLinux Minor version number

The step sets AlmaLinux `version_minor` according to set on inputs `version_major`.

#### Step: Prepare date stamp

Generates `date_stamp` in format *YYYYMMDD*. It is used in image tags.

#### Step: Generate list of Docker images to use as base name for tags

Generates `env.IMAGE_NAMES` for each registry including image type like: `docker.io/***/8-minimal quay.io/***/8-minimal`

#### Step: Enable containerd image store on Docker Engine

Modifies the /etc/docker/daemon.json as:
```json
"features":
{
    "containerd-snapshotter": true
}
```
Restarts `docker` service to get new image store working.
The successful switch is printed in the docker info:
```json
[[driver-type io.containerd.snapshotter.v1]]
```

#### Step: Checkout *container-images*, branch 'main'

Checkouts *container-images* into branch 'main'. The repository directory is located at `/home/runner/work/container-images/container-images`. Please note, the only last commit is checked out.
The [actions/checkout@v4](https://github.com/actions/checkout/) is used.

#### Step: Checkout *container-images*, branch 'docker-library', path 'docker-library'

Checkouts *container-images* into branch 'docker-library'. The repository directory is located at `/home/runner/work/container-images/docker-library`.
The [actions/checkout@v4](https://github.com/actions/checkout/) is used.

#### Step: Set up QEMU

Installs [QEMU](https://github.com/qemu/qemu) static binaries. The [docker/setup-qemu-action@v3](https://github.com/docker/setup-qemu-action) is used. The QEMU static binaries are required to build different platforms within one machine.

#### Step: Set up Docker Buildx

Sets up Docker [Buildx](https://github.com/docker/buildx). It uses [docker/setup-buildx-action@v3](https://github.com/docker/setup-buildx-action)

#### Step: Login to Docker.io

The [docker/login-action@v3](https://github.com/docker/login-action) is used. The following secrets are used:

*production* mode:
- DOCKERHUB_USERNAME
- DOCKERHUB_TOKEN

*testing* mode:
- TEST_DOCKERHUB_USERNAME
- TEST_DOCKERHUB_TOKEN

#### Step: Login to Quay.io

The [docker/login-action@v3](https://github.com/docker/login-action) is used. The following secrets are used:

*production* mode:
- QUAY_IO_USERNAME
- QUAY_IO_CLI_PASSWORD

*testing* mode:
- TEST_QUAY_IO_USERNAME
- TEST_QUAY_IO_CLI_PASSWORD

#### Step: Login to Ghcr.io

The [docker/login-action@v3](https://github.com/docker/login-action) is used. The following secrets are used:

*production* mode:
- GIT_HUB_USERNAME
- GIT_HUB_TOKEN

*testing* mode:
- TEST_GIT_HUB_USERNAME
- TEST_GIT_HUB_TOKEN

#### Step: Generate tags and prepare metadata to build and push

The [docker/metadata-action@v5](https://github.com/docker/metadata-action) is used to generate tags, labels and annotations for images. Here is an example of AlmaLinux 8 minimal image's tags:
```json
"tags": [
          "docker.io/***/8-minimal:latest",
          "docker.io/***/8-minimal:8",
          "docker.io/***/8-minimal:8.9",
          "docker.io/***/8-minimal:8.9-20240319",
          "quay.io/***/8-minimal:latest",
          "quay.io/***/8-minimal:8",
          "quay.io/***/8-minimal:8.9",
          "quay.io/***/8-minimal:8.9-20240319",
        ],
```
#### Step: Build images

The [docker/build-push-action@v5](https://github.com/docker/build-push-action) is used to build images. This step builds the images from corresponded [`Containerfile`](https://github.com/AlmaLinux/container-images/tree/main/Containerfiles), for specified `env.platforms` and using tags from the previous step. After the successful building, the images are loaded into docker, and not yet pushed because they need to be tested. AlmaLinux 8 minimal images buildx looks like:
```sh
/usr/bin/docker buildx build --file ./Containerfile.minimal ... \
 --platform linux/amd64,linux/ppc64le,linux/s390x,linux/arm64 \
 --provenance false ... \
 --tag docker.io/***/8-minimal:latest --tag docker.io/***/8-minimal:8 --tag docker.io/***/8-minimal:8.9 --tag docker.io/***/8-minimal:8.9-20240319 --tag quay.io/***/8-minimal:latest --tag quay.io/***/8-minimal:8 --tag quay.io/***/8-minimal:8.9 --tag quay.io/***/8-minimal:8.9-20240319 \
 --load \
 --metadata-file /home/runner/work/_temp/docker-actions-toolkit-*/metadata-file \
 https://github.com/***/container-images.git#270a6d3fd433cfa6c3e1fff5896a92d1ae2896be:Containerfiles/8
```
`provenance: false` is to disable the [Provenance attestations](https://docs.docker.com/build/attestations/slsa-provenance/) as Quay.io registry doesn't support such kind of images data.

#### Step: Test images

As every image is loaded into docker, so it can be tested separately for each type and platform. Docker run images by digest:
```sh
docker run --platform=${platform} ${{ steps.build-images.outputs.digest }}
```

#### Step: Push images to Client Library

The [docker/build-push-action@v5](https://github.com/docker/build-push-action) is used. This step pushes built images into *Client Library*. The options are the same as for **Build images** step.


#### Step: Change date stamp in Containerfile (default and minimal only)

> The step is skipped if image type is not 'default' or 'minimal'

The step changes (*# Tags* with date stamp) corresponded [`Containerfiles/*/Containerfile.*`](https://github.com/AlmaLinux/container-images/tree/docker-library), which Docker will use to build images for the *Official Library*. An example is for AlmaLinux 8 minimal `Containerfiles/8/Containerfile.minimal` file:
```docker
# Tags: 8-minimal, 8.9-minimal, 8.9-minimal-20240319

FROM quay.io/almalinuxorg/8-minimal:8.9-20240319
```
The change indicates that new `default` and/or `minimal` container image was pushed to *Client Library* and the same should be requested to build by Docker. The change will later be committed into `docker-file` branch.

#### Step: Upload changed *Containerfiles/*/Containerfile.**

> The step is skipped if image type is not 'default' or 'minimal'

The step uses [actions/upload-artifact@v4](https://github.com/actions/upload-artifact) to store as artifact changed on previous step Containerfile. The artifact is named against `image_type`, like `containerfiles-image_types`.

Artifacts are used to transfer files between different jobs of the same workflow. The artifact is zip archive of the file without file-path included.

That is also possible to download artifacts via GitHub Action's web interface.

### Job: Collect and save changed Containerfile(s) used by Docker Official Library

> The job is skipped if image type is not 'default' or 'minimal'

#### Step: Checkout *container-images*, branch 'docker-library'

Checkouts *container-images* into branch 'docker-library'. The repository directory is located at `/home/runner/work/container-images/container-images`.
The [actions/checkout@v4](https://github.com/actions/checkout/) is used.

#### Step: Download changed Containerfiles

Uses [actions/download-artifact@v4](https://github.com/actions/download-artifact) to download multiple (`merge-multiple: true`) artifacts with changed Containerfiles. The files are saved into the `Containerfiles/version_major/` directory.

#### Step: Commit and push *Containerfiles/version_major/Containerfile.** changes

> The step is skipped if '*Push to production registries*' is not checked (`inputs.production` set to `false`.)

Uses [EndBug/add-and-commit@v9](https://github.com/marketplace/actions/add-commit) to commit and push Containerfiles, which was downloaded on the previous step, and changed by the previous job.

The commit message is:
```yaml
AlmaLinux ${{ inputs.version_major }} image build as of ${{ needs.build.outputs.date_stamp }} (with ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}).
```
It includes AlmaLinux version major, image build date, and reference to this Github Action.

## Publish images to the Docker Library

Tree illustration of the workflow Jobs and Steps for AlmaLinux 9 minimal image:
```
Publish images to the Docker Library
│
├── 8 default definition preparing
.
.
.
├── 8 minimal definition preparing
.
.
.
├── 9 default definition preparing
.
.
.
├── 9 minimal definition preparing
│   ├── Set up job
│   ├── Checkout container-images, branch 'docker-library'
│   ├── Checkout official-images, branch 'master'
│   ├── Get need data for the definition
│   ├── Render the definition
│   ├── Upload the definition for 9 minimal
│   ├── Post Checkout official-images, branch 'master'
│   ├── Post Checkout container-images, branch 'docker-library'
│   └── Complete job
│
└── Create Pull Request with the new definition file
    ├── Set up job
    ├── Checkout official-images, branch 'master'
    ├── Sync official-images with docker-library/official-images
    ├── Download all definitions
    ├── Create head of official-images/library/almalinux
    ├── Merge definitions into official-images/library/almalinux
    ├── Prepare date stamp
    ├── Prepare time stamp
    ├── Commit and push official-images/library/almalinux
    ├── Create Pull Request for official-images/library/almalinux
    ├── Post Checkout official-images, branch 'master'
    └── Complete job
```

### Inputs

The workflows inputs are:
- `pr` - boolean *'Publish to the Docker Official Library'* with the default value `true` (checked). The input indicates whether to create Pull Request to Docker with AlmaLinux *Definition File*.

- `draft` - boolean '*Draft Pull Request*' with the default value `false` (not checked). The input indicates whether the [Pull Request is draft](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests#draft-pull-requests) or it is ready to review.

### Job: *version_major* *image_types* definition preparing

Job iterates (using matrix) with AlmaLinux all `version_major`, and `image_types` (`default` and `minimal`). Multiple jobs run simultaneously for each of versions and for each of image types.

#### Step: Checkout *container-images*, branch 'docker-library'

The [actions/checkout@v4](https://github.com/actions/checkout/) checkouts *container-images* into branch 'docker-library'. The repository directory is located at `/home/runner/work/container-images/container-images`. All commits for the branch are checkout with `fetch-depth: 0`.

#### Step: Checkout *official-images*, branch 'master'

The [actions/checkout@v4](https://github.com/actions/checkout/) checkouts *container-images* into branch 'master'. The repository directory is located at `/home/runner/work/container-images/official-images`.

That's your fork of [docker-library/official-images](https://github.com/docker-library/official-images) repository.

#### Step: Get need data for the definition

The step is written in bash. It reads the *# Tags:* string and the commit hash of `Containerfiles/${{ matrix.version_major }}/Containerfile.image_types` file changed by the **Build, test and push** workflow. `env.tags` and `env.last_commit` are exported. These data will be used to generate part of *Definition File*.

#### Step: Render the definition

The [chuhlomin/render-template@v1](https://github.com/marketplace/actions/render-template) generates from the `docker-library-definition.tmpl` using data (`env.tags`, `env.last_commit`, `matrix.version_major` and `matrix.image_types`) file `official-images/library/almalinux.version_major.image_types`. The file is a part of Docker Library *Definition File*.

#### Step: Upload the definition for *version_major* *image_types*

The step uses [actions/upload-artifact@v4](https://github.com/actions/upload-artifact) to store as artifact generated on previous step `official-images/library/almalinux.version_major.image_types`. The artifact is named against `version_major` and `image_type`, like `definition-version_major.image_types`.

Artifacts are used to transfer files between different jobs of the same workflow. The artifact is zip archive of the file without file-path included.

That is also possible to download artifacts via GitHub Action's web interface.

### Job: Create Pull Request with the new definition file

> The job is skipped if the *'Publish to Docker Official Library'* isn't checked (`inputs.pr` set into `false`)

#### Step: Checkout *official-images*, branch 'master'

The [actions/checkout@v4](https://github.com/actions/checkout/) checkouts *container-images* into branch 'master'. The repository directory is located at `/home/runner/work/container-images/official-images`.

That's your fork of [docker-library/official-images](https://github.com/docker-library/official-images) repository.

#### Step: Sync *official-images* with upstream

The step is written in bash. It adds the [official-images upstream](https://github.com/docker-library/official-images) repository, set local 'master' branch to track corresponded from the upstream, and tries to rebase with the 'upstream/master'.

#### Step: Download all definitions

Uses [actions/download-artifact@v4](https://github.com/actions/download-artifact) to download multiple (`merge-multiple: true`) artifacts with generated definitions. The files are saved into the `official-images/library/` directory.

#### Step: Create head of *official-images/library/almalinux*

Creates heading for the Docker *Definition File*.

#### Step: Merge definitions into *official-images/library/almalinux*

The step is written in bash. It appends the `official-images/library/almalinux` *Definition File* with downloaded on the previous step definitions `official-images/library/almalinux.version_major.image_types`.

#### Step: Prepare date stamp

Generates `date_stamp` in format *YYYYMMDD*. It is used in commit message and pull request title.

#### Step: Prepare time stamp

Generates `time_stamp` in format *HH:MM:SS*. It is used in commit message and pull request title.

#### Step: Commit and push *official-images/library/almalinux*

Uses [EndBug/add-and-commit@v9](https://github.com/marketplace/actions/add-commit) to commit and push the generated *Definition File*.

The commit message is:
```yaml
Almalinux auto-update - ${{ env.date_stamp }}.
```

#### Step: Create Pull Request for *official-images/library/almalinux*

The step is written in bash. It uses Github CLI to create Pull Request for the `official-images/library/almalinux` *Definition File* from your fork and to [docker-library/official-images](https://github.com/docker-library/official-images) repository.

The Pull Request will be draft if the `draft` input is checked. When ready, [mark the request as ready for review](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/changing-the-stage-of-a-pull-request#marking-a-pull-request-as-ready-for-review).