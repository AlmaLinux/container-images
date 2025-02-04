# Overview

The repository provides scripts and workflows to:

- Build AlmaLinux Container Images (Docker Images)
- Test these images
- Push them into different registries, the *Client Library*:
    - [Docker.io](https://hub.docker.com)
    - [Quay.io](https://quay.io)
    - [GitHub Packages](https://github.com/features/packages)
- Extract root filesystem (RootFS) from default and minimal images
- Publish the images to the [Docker *Official Library*](https://hub.docker.com/u/library).

These Container Images can be used with all [OCI complaint](https://opencontainers.org/) container runtime environments such as Docker, Podman and Kubernetes as well as serve as drop-in replacements for CentOS images as they reach [End of Life](https://centos.org/centos-linux-eol/).

## Requirements/Prerequisites

Personal, Organization or Enterprise account on GitHub is the only requirement. Please read more about [accounts on GitHub](https://docs.github.com/en/enterprise-server@3.10/get-started/learning-about-github/types-of-github-accounts).

## The idea

 The project utilizes [GitHub Actions](https://github.com/features/actions) to provide public, transparent and fast workflows that are easy to understand, use and modify.

There are two workflows on GitHub Actions designed to achieve the idea:
- Build, test, push all types of container images into the *Client Library*, and extract RootFS from `default` and `minimal` images.
- Use these RootFS to request Docker to create images for the *Official Library*.

You can read more about how the workflows work in the [section](#workflows-jobs-and-steps) below.

The AlmaLinux ***Client Library*** includes the following registries/organizations:
- [Docker.io/almalinux](https://hub.docker.com/u/almalinux) (Sponsored OSS)
- [Quay.io/almalinuxorg](https://quay.io/organization/almalinuxorg)
- [Ghcr.io/AlmaLinux](https://github.com/orgs/AlmaLinux/packages)

The AlmaLinux [***Official Library***](https://hub.docker.com/_/almalinux) is maintained by Docker.

## Containerfiles

Each image pushed to the *Client Library* is built from a corresponding [Containerfile](https://github.com/AlmaLinux/container-images/tree/main/Containerfiles) that is a unique file for each AlmaLinux release and configuration type: `base`, `default`, `init`, `micro`, `minimal`.
These files match [Dockerfile](https://docs.docker.com/reference/dockerfile/) standard and contain commands and instructions on how to install AlmaLinux's whole root filesystem in them.

Images for the *Docker Official* Library are built using other Dockerfiles that are also designed for each AlmaLinux release but only `default` and `minimal` types:
- 8 [default](https://github.com/yuravk/container-images/tree/8/default) and [minimal](https://github.com/yuravk/container-images/tree/8/minimal) per platform;
- 9 [default](https://github.com/yuravk/container-images/tree/9/default) and [minimal](https://github.com/yuravk/container-images/tree/9/minimal) per platform.
These Dockerfiles are to build images from scratch using platform's corresponding RootFS.

## What Container Images are built

### AlmaLinux releases

Container images are built for AlmaLinux OS 8 and 9. The Major version of the release must be set for the **Build, Test and Push** workflow. The Minor version is automatically set by the workflow as *the latest*.

**Publish Images** workflow pushes build requests to the Docker also for both AlmaLinux releases, 8 and 9.

### Image configuration types

AlmaLinux container images types match [Red Hat Universal Base Image](https://catalog.redhat.com/software/base-images):
- `base`
- `default` (the image is also available via the Docker *Official Library*)
- `init`
- `micro`
- `minimal` (the image is also available via the Docker *Official Library*)

### Supported platforms/architectures

**Build, Test and Push** workflow builds container images of the following platforms simultaneously with `docker buildx`. They result in the following machine hardware names (`uname -m`):

| docker platform | hardware name |
| --------------- | ------------- |
| linux/amd64     | x86_64        |
| linux/ppc64le   | ppc64le       |
| linux/s390x     | s390x         |
| linux/arm64     | aarch64       |

The [**containerd image store store**](https://docs.docker.com/storage/containerd/) for Docker Engine together with `buildx` are used to build and push multiple platforms at once.

### Repositories

The following *repositories* are created on all registries ([Docker.io/almalinux](https://hub.docker.com/u/almalinux), [Quay.io/almalinuxorg](https://quay.io/organization/almalinuxorg), [Ghcr.io/AlmaLinux](https://github.com/orgs/AlmaLinux/packages)) for all supported images types:

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

The `/almalinux` *repository* includes the `latest` tag for AlmaLinux release 9.x only.

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

2. Branch for AlmaLinux release '8'
```sh
.
├── docker-library-definition.tmpl
│
├── default
│   ├── amd64
│   │   ├── Dockerfile
│   │   └── almalinux-8-default-amd64.tar.gz
│   ├── arm64
│   │   ├── Dockerfile
│   │   └── almalinux-8-default-arm64.tar.gz
│   ├── ppc64le
│   │   ├── Dockerfile
│   │   └── almalinux-8-default-ppc64le.tar.gz
│   └── s390x
│       ├── Dockerfile
│       └── almalinux-8-default-s390x.tar.gz
└── minimal
    ├── amd64
    │   ├── Dockerfile
    │   └── almalinux-8-minimal-amd64.tar.gz
    ├── arm64
    │   ├── Dockerfile
    │   └── almalinux-8-minimal-arm64.tar.gz
    ├── ppc64le
    │   ├── Dockerfile
    │   └── almalinux-8-minimal-ppc64le.tar.gz
    └── s390x
        ├── Dockerfile
        └── almalinux-8-minimal-s390x.tar.gz
```

3. Branch for AlmaLinux release '9'
```sh
.
├── docker-library-definition.tmpl
│
├── default
│   ├── amd64
│   │   ├── Dockerfile
│   │   └── almalinux-9-default-amd64.tar.gz
│   ├── arm64
│   │   ├── Dockerfile
│   │   └── almalinux-9-default-arm64.tar.gz
│   ├── ppc64le
│   │   ├── Dockerfile
│   │   └── almalinux-9-default-ppc64le.tar.gz
│   └── s390x
│       ├── Dockerfile
│       └── almalinux-9-default-s390x.tar.gz
└── minimal
    ├── amd64
    │   ├── Dockerfile
    │   └── almalinux-9-minimal-amd64.tar.gz
    ├── arm64
    │   ├── Dockerfile
    │   └── almalinux-9-minimal-arm64.tar.gz
    ├── ppc64le
    │   ├── Dockerfile
    │   └── almalinux-9-minimal-ppc64le.tar.gz
    └── s390x
        ├── Dockerfile
        └── almalinux-9-minimal-s390x.tar.gz
```

### Workflow **.yml* files

The [`.github/workflows/build-test-push.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/build-test-push.yml) workflow is used to **Build, Test and Push** images to the *Client Library*, and extract RootFS:
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

The
[`.github/workflows/publish-docker-library.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/publish-docker-library.yml) workflow is used to **Publish Images** to the Docker *Official Library*:
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

Both workflows are triggered manually by the [**workflow_dispatch**](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#workflow_dispatch) event of GitHub Actions.

### Sourced *Containerfiles*

This [`Containerfiles/9/Containerfile.minimal`](https://github.com/AlmaLinux/container-images/blob/main/Containerfiles/9/Containerfile.minimal) file is a Containerfile example for AlmaLinux release 9 and `minimal` type used to build container image for the *Client Library*:
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

This [`minimal/amd64/Dockerfile`](https://github.com/yuravk/container-images/blob/9/minimal/amd64/Dockerfile) file is a Dockerfile example for AlmaLinux release 9 `minimal` type and `amd64` (`x86_64`) platform used to build container image for the Docker *Official Library*:
```Dockerfile
# Tags: minimal, 9-minimal, 9.3-minimal, 9.3-minimal-20231124
FROM scratch
ADD almalinux-9-minimal-amd64.tar.xz /

CMD ["/bin/bash"]
```

### Template file for Docker *Library Definition*

The Docker *Official Library* uses [Definition File](https://github.com/docker-library/official-images/blob/master/library/almalinux) to request building of official images. Changing the file triggers a new image(s) building on the Docker side. The [`docker-library-definition.tmpl`](https://github.com/yuravk/container-images/blob/docker-library/docker-library-definition.tmpl) template is used to generate the Definition file:
```yaml
Tags: {{ .tags }}
GitFetch: refs/heads/{{ .version_major }}
GitCommit: {{ .commit_hash }}
amd64-Directory: {{ .image_type }}/amd64/
arm64v8-Directory: {{ .image_type }}/arm64/
ppc64le-Directory: {{ .image_type }}/ppc64le/
s390x-Directory: {{ .image_type }}/s390x/
Architectures: amd64, arm64v8, ppc64le, s390x
```

# How to contribute/help and customize workflow(s)

## Fork GitHub repositories

Fork the following repositories:
- [**container-images**](https://github.com/AlmaLinux/container-images), you will need the `main`, the `8` and the `9` branches.
- [**docker-library**](https://github.com/docker-library/official-images)

Read more about GitHub [forks here](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo).

❗ Please, note, that you won't be able to create a Pull Request to this repository as only AlmaLinux organization members have access to do it.

## Set Action's secrets

To set secrets needed for this repository, go to your GitHub account **Settings** -> expand **Secrets and variables** (located under the **Security** section) -> select **Actions**. Read more about [Github Secrets in Actions](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions).

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

When creating a new personal access token on **GitHub**, please, select:
- *write:packages* scope, to allow packages uploading to GitHub Package Registry;
- *admin:org* scope, to allow Pull Request creation to [docker-library](https://github.com/docker-library/official-images).

## Change registries list

On needed registries create your own accounts in case you don't have any as you won't be able to use AlmaLinux accounts.

According to your list of needed registries and knowing your user names edit the [`.github/workflows/build-test-push.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/build-test-push.yml) workflow file (branch `master`):
```yaml
registries: 'docker.io/<user_name>, quay.io/<user_name>, ghcr.io/<user_name>'
```
Separate the registries with commas.

`<user_name>` - is your user name on the specific registry.

## Change platforms list

If you don't need to build images for all platforms, you can edit the list of platforms to meet your needs in the [`.github/workflows/build-test-push.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/build-test-push.yml) workflow file (branch `master`):

```yaml
platforms: 'linux/amd64, linux/ppc64le, linux/s390x, linux/arm64'
```
Separate the platforms with commas.

## Change image types list

Edit the [`.github/workflows/build-test-push.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/build-test-push.yml) workflow file (branch `master`) to change type of images which are built:

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
Where `<type_name>` is the name of your image type.

Default are: *base*, *default*, *init*, *micro*, *minimal*

## To bump AlmaLinux release (*Major* number)

If a new AlmaLinux major release is available, edit the [`.github/workflows/build-test-push.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/build-test-push.yml) workflow file (branch `master`), to set this major version:

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
Where `<version_latest>` is an AlmaLinux version major version, for example `10`.

## To bump AlmaLinux release (*Minor* number)

If a new AlmaLinux minor release is available, edit the [`.github/workflows/build-test-push.yml`](https://github.com/AlmaLinux/container-images/blob/main/.github/workflows/build-test-push.yml) workflow file (branch `master`) to set it. To do so, you need the "DeployPrepare AlmaLinux Minor version number" step:

```sh
          case ${{ inputs.version_major }} in
            8)
              version_minor="<8_minor>" ;;
            9)
              version_minor="<9_minor>"  ;;
            10)
              version_minor="<10_minor>" ;;

```

Where `<8_minor>`, `<9_minor>`, `<10_minor>` are AlmaLinux's corresponding minor versions.

For example Minors are `10`, `4` or `1` for new **8.10**, **9.4** or **10.1** versions respectively.

## Restrictions

❗ Only AlmaLinux organization members have access to create Pull Requests and publish container images into the Docker *Official Library*.

❗ **Build, test and push to the Client Library** workflow will work only for your *Client Library*, but not for AlmaLinux-owned ones.

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
    │   ├── Generate list of Docker images to use as base name for tags
    │   ├── Enable containerd image store on Docker Engine
    │   ├── Checkout _container-images, branch 'main'
    │   ├── Checkout _container-images, branch '9', path '9'
    │   ├── Set up QEMU
    │   ├── Set up Docker Buildx
    │   ├── Login to Docker.io
    │   ├── Login to Quay.io
    │   ├── Login to Ghcr.io
    │   ├── Generate tags and prepare metadata to build and push
    │   ├── Build images
    │   ├── Test images
    │   ├── Push images to Client Library
    │   ├── Extract RootFS (default and minimal only)
    │   ├── Change date stamp in Dockerfile (default and minimal only)
    │   ├── Commit and push minimal/*/* Dockerfile and RootFS (branch 9)"
    │   ├── Post Push images to Client Library
    │   ├── Post Build images
    │   ├── Post Login to Ghcr.io
    │   ├── Post Set up Docker Buildx
    │   ├── Post Checkout _container-images, branch '9', path '9'
    │   ├── Post Checkout _container-images, branch 'main'
    │   └── Complete job
    │
    └── Optimize size of repository
        ├── Checkout almalinux/container-images, branch '9', path '9'
        ├── Optimize size of branch the '9'
        └── Commit and push almalinux/container-images, branch '9'
```

### Inputs

The workflow inputs are:
- `production` - boolean '*Push to production registries*' with the default value `true` (checked). Container images are pushed into the production *Client Library*: [Docker.io/almalinux](https://hub.docker.com/u/almalinux), [Quay.io/almalinuxorg](https://quay.io/organization/almalinuxorg) and [Ghcr.io/AlmaLinux](https://github.com/orgs/AlmaLinux/packages). Otherwise, images are pushed into the testing *Client Library*: [Quay.io/almalinuxautobot](https://quay.io/organization/almalinuxautobot)

- `version_major` - dropdown 'AlmaLinux major version' with the default value `9`. This is a major number of AlmaLinux version to build images for.

- Checklist of image types: *base*, *default*, *init*, *micro*, *minimal*. At least one should be checked.

### Job: Deploy *version_major* *image_types* images

Job proceeds to input `version_major` and iterates with selected `image_types` using matrix. Multiple jobs run simultaneously for each image type.

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
Restarts the `docker` service to get a new image store working.
The successful switch is printed in the docker info:
```json
[[driver-type io.containerd.snapshotter.v1]]
```

#### Step: Checkout *container-images*, branch 'main'

Checkouts *container-images* into branch 'main'. The repository directory is located at `/home/runner/work/container-images/container-images`. Please note, the only last commit is checked out.
The [actions/checkout@v4](https://github.com/actions/checkout/) is used.

#### Step: Checkout *container-images*, branch '${version_major}', path '${version_major}'

Checkouts *container-images* into branch '${version_major}'. The repository directory is located at `/home/runner/work/container-images/${version_major}`.
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

The [docker/build-push-action@v5](https://github.com/docker/build-push-action) is used to build images. This step builds the images from corresponding [`Containerfile`](https://github.com/AlmaLinux/container-images/tree/main/Containerfiles), for specified `env.platforms` and uses tags from the previous step. After the successful building, the images are loaded into docker, but not pushed yet as they need to be tested first. AlmaLinux 8 minimal images `buildx` looks like this:
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

Every image can be tested separately for each type and platform as each image is loaded into docker. Docker run images "by digest":
```sh
docker run --platform=${platform} ${{ steps.build-images.outputs.digest }}
```

#### Step: Push images to Client Library

The [docker/build-push-action@v5](https://github.com/docker/build-push-action) is used. This step pushes built images into *Client Library*. The options are the same as for **Build images** step.

#### Step: Extract RootFS (default and minimal only)

❗ Skip this step if the image type is not 'default' or 'minimal'.

The step is to extract RootFS from existing image's blobs:
- uses`docker save` to produce a tarred repository and save it to the "tar file". Unpack the "tar file" to get blobs.
- Prepares the "temporary Dockerfile" to build image based on RootFS.
```Dockerfile
    FROM scratch
    ADD rootfs.tar.gz /
    CMD ["/bin/bash"]
```
- Loops blobs to find all zipped files that are RootFS for a particular architecture.
- with `docker build`, builds an image from the "temporary Dockerfile".
- with `docker run`, runs the image and query `almalinux-release` package's *architecture*.
- Maps found *architecture* to the corresponding *platform*.
- Copes the "taken RootFS" into corresponded .tar.xz (like `almalinux-9-default-amd64.tar.xz`)

#### Step: Change date stamp in Dockerfile (default and minimal only)

❗ Skip this step if the image type is not 'default' or 'minimal'.

The step changes (*# Tags* with date stamp) in corresponded `${images_type}/${platform}/Docker file` for AlmaLinux [release 8](https://github.com/yuravk/container-images/tree/8) an [release 9](https://github.com/yuravk/container-images/tree/9), which Docker will use to build images for the *Official Library*. An example is for AlmaLinux 9 minimal amd64 [`minimal/amd64/dockerfile`](https://github.com/yuravk/container-images/blob/9/minimal/amd64/Dockerfile) file:
```docker
# Tags: 8-minimal, 8.9-minimal, 8.9-minimal-20240319
FROM scratch
ADD almalinux-9-minimal-amd64.tar.xz /

CMD ["/bin/bash"]
```
The change indicates that a new `default` and/or `minimal` container image was pushed to the *Client Library* and should be requested to be built by Docker. The change will later be committed to the `8` or `9` branch.

> It will try to pull recent changes (before push) with `--rebase --autostash`

#### Step: Commit and push ${image_types }/*/* Dockerfile and RootFS (branch ${version_major })"

❗ Skip this step if the image type is not 'default' or 'minimal'.

❗ The step is skipped if '*Push to production registries*' is not checked (`inputs.production` set to `false`.)

Uses [EndBug/add-and-commit@v9](https://github.com/marketplace/actions/add-commit) to commit and push Dockerfile and RootFS, which were changed and extracted on the previous steps.

The commit message is:
```yaml
AlmaLinux ${version_major}-${images_type} image build as of ${date_stamp} (with ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}).
```
It includes the AlmaLinux version major, image type, build date, and reference to this GitHub Action.

### Job: Optimize size of repository

❗ Skip the job if the image type is not 'default' or 'minimal', or '*Push to production registries*' is not checked (`inputs.production` set to `false`.)

#### Step: Checkout almalinux/container-images, branch '${version_major}', path '${version_major}'

The [actions/checkout@v4](https://github.com/actions/checkout/) checkouts *container-images* into branch '${version_major}'. The repository directory is located at `/home/runner/work/container-images/container-images` and its subdirectory is named '${version_major}'. All commits for the branch are checkout with `fetch-depth: 0`.

#### Step: Optimize size of branch the '${version_major}'

This step is written in bash and is designed to:
- preserves 'default' and 'minimal' folders with Dockerfiles and RootFSs into `../tmp-${date_stamp}` folder
- checkouts into new local 'tmp' branch
- removes local ${version_major} branch
- checkouts into orphan ${version_major} branch
- restores 'default' and 'minimal' folders into the orphan branch placeholder

#### Step: Commit and push almalinux/container-images, branch '${version_major}'

Uses [EndBug/add-and-commit@v9](https://github.com/marketplace/actions/add-commit) to commit and push Dockerfiles and RootFSs which were prepared on previous step.
The following options are used to push:
- `--force` - to rewrite history
- `--set-upstream origin ${version_major}` - to set upstream branch (as new one is orphan)

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
│   ├── Checkout container-images, branch '9'
│   ├── Checkout official-images, branch 'master'
│   ├── Get need data for the definition
│   ├── Render the definition
│   ├── Upload the definition for 9 minimal
│   ├── Post Checkout official-images, branch 'master'
│   ├── Post Checkout container-images, branch '9'
│   └── Complete job
│
└── Create Pull Request with the new definition file
    ├── Set up job
    ├── Checkout official-images, branch 'master'
    ├── Sync official-images with docker-library/official-images, branch 'master'
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

The workflow inputs are:
- `pr` - boolean '*Publish to the Docker Official Library*' with the default value `true` (checked). The input indicates whether to create a Pull Request to Docker with AlmaLinux *Definition File*.

- `draft` - boolean '*Draft Pull Request*' with the default value `false` (not checked). The input indicates whether the [Pull Request is draft](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests#draft-pull-requests) or it is ready to review.

### Job: *version_major* *image_types* definition preparing

Job iterates (using matrix) with AlmaLinux all `version_major`, and `image_types` (`default` and `minimal`). Multiple jobs run simultaneously for each of the versions and each of the image types.

#### Step: Checkout *container-images*, branch '${version_major}'

The [actions/checkout@v4](https://github.com/actions/checkout/) checkouts *container-images* into branch '${version_major}'. The repository directory is located at `/home/runner/work/container-images/container-images`. All commits for the branch are checkout with `fetch-depth: 0`.

#### Step: Checkout *official-images*, branch 'master'

The [actions/checkout@v4](https://github.com/actions/checkout/) checkouts *container-images* into branch 'master'. The repository directory is located at `/home/runner/work/container-images/official-images`.

That's your fork of [docker-library/official-images](https://github.com/docker-library/official-images) repository.

#### Step: Get need data for the definition

The step is written in bash. It reads the *# Tags:* string and the commit hash of `Containerfiles/${{ matrix.version_major }}/Containerfile.image_types` file changed by the **Build, test and push** workflow. `env.tags` and `env.last_commit` are exported. These data will be used to generate part of *Definition File*.

#### Step: Render the definition

The [chuhlomin/render-template@v1](https://github.com/marketplace/actions/render-template) generates from the `docker-library-definition.tmpl` using data (`env.tags`, `env.last_commit`, `matrix.version_major` and `matrix.image_types`) file `official-images/library/almalinux.version_major.image_types`. The file is a part of Docker Library *Definition File*.

#### Step: Upload the definition for *version_major* *image_types*

The step uses [actions/upload-artifact@v4](https://github.com/actions/upload-artifact) to store an artifact generated in the previous step `official-images/library/almalinux.version_major.image_types`. The artifact is named against `version_major` and `image_type`, following the pattern: `definition-${version_major}.${image_types}`.

Artifacts are used to transfer files between different jobs of the same workflow. The artifact is a zip archive of the file without file-path included.

It is also possible to download artifacts via GitHub Action's web interface.

### Job: Create Pull Request with the new definition file

> The job is skipped if the *'Publish to Docker Official Library'* isn't checked (`inputs.pr` set into `false`)

#### Step: Checkout *official-images*, branch 'master'

The [actions/checkout@v4](https://github.com/actions/checkout/) checkouts *container-images* into branch 'master'. The repository directory is located at `/home/runner/work/container-images/official-images`.

That's your fork of [docker-library/official-images](https://github.com/docker-library/official-images) repository.

#### Step: Sync *official-images* with upstream

The step is written in bash. It uses GitHub CLI to sync the [docker-library-official-images](https://github.com/AlmaLinux/docker-library-official-images) with the [official-images upstream](https://github.com/docker-library/official-images) repository.

#### Step: Download all definitions

Uses [actions/download-artifact@v4](https://github.com/actions/download-artifact) to download multiple (`merge-multiple: true`) artifacts with generated definitions. The files are saved into the `official-images/library/` directory.

#### Step: Create head of *official-images/library/almalinux*

Creates heading for the Docker *Definition File*.

#### Step: Merge definitions into *official-images/library/almalinux*

The step is written in bash. It appends the `official-images/library/almalinux` *Definition File* with downloaded on the previous step definitions `official-images/library/almalinux.version_major.image_types`.

#### Step: Prepare date stamp

Generates `date_stamp` in the *YYYYMMDD* format. It is used in the commit message and pull request title.

#### Step: Prepare time stamp

Generates `time_stamp` in the *HH:MM:SS* format. It is used in the commit message and pull request title.

#### Step: Commit and push *official-images/library/almalinux*

Uses [EndBug/add-and-commit@v9](https://github.com/marketplace/actions/add-commit) to commit and push the generated *Definition File*.

The commit message is:
```yaml
AlmaLinux auto-update - ${{ env.date_stamp }} ${{ env.time_stamp }}.
```
> It will try to pull recent changes (before push) with `--rebase --autostash`

#### Step: Create Pull Request for *official-images/library/almalinux*

The step is written in bash. It uses Github CLI to create a Pull Request for the `official-images/library/almalinux` *Definition File* from your fork and to [docker-library/official-images](https://github.com/docker-library/official-images) repository.

The Pull Request will be drafted if the `draft` input is checked. When ready, [mark the request as ready for review](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/changing-the-stage-of-a-pull-request#marking-a-pull-request-as-ready-for-review).