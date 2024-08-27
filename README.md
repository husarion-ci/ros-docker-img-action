# ros-docker-img-action

Publish docker images with tags containing ROS distro, package version, release date and manage stable releases.

## Examples

Create `your-repo-docker/.github/workflows/build-docker.img.yaml` and use one of the following templates

### Minimal workflow

It will publish the images:
- `husarion/your-repo:humble-<tag>-<date>`
- `husarion/your-repo:humble-nightly`

```yaml
name: Build/Publish Docker Image (development only)

on: 
  workflow_dispatch:

  repository_dispatch:
    types: [rebuild]
  pull_request:
    types:
      - closed
      - opened

jobs:
  build:
    runs-on: ubuntu-20.04

    steps:

      - name: Checkout
        uses: actions/checkout@v2

      - name: Build Docker Image
        uses: husarion-ci/ros-docker-img-action@v0.1
        with:
          dockerhub_username: ${{ secrets.DOCKERHUB_USERNAME }}
          dockerhub_token:  ${{ secrets.DOCKERHUB_TOKEN }}
```

### Full workflow

It will publish the images:

1. On `repository_dispatch` or `repository_dispatch` or `workflow_dispatch` with `build_type=development`
- `husarion/your-repo:<ros-distro>-<tag>-<date>`
- `husarion/your-repo:<ros-distro>-nightly`

2. On `repository_dispatch` with `build_type=stable`
- `husarion/your-repo:<ros-distro>-<tag>-<date>-stable`
- `husarion/your-repo:<ros-distro>-<tag>-<date>`
- `husarion/your-repo:<ros-distro>-<tag>`
- `husarion/your-repo:<ros-distro>-nightly`
- `husarion/your-repo:<ros-distro>`

3. On `repository_dispatch` on different branch than `main` (eg. `my-feature-1`)
- `husarion/your-repo:<ros-distro>-my-feature-1`

```yaml
name: Build/Publish Docker Image 

on: 
  workflow_dispatch:
    inputs:
      build_type:
        description: "Is it a \"development\" or a \"stable\" release?"
        required: true
        default: 'development'
        type: choice
        options:
          - development
          - stable
      target_distro:
        description: "In case of \"stable\" release specify the ROS distro of the existing docker image (eg. humble)"
        type: string
        default: "ardent"
      target_release:
        description: "In case of \"stable\" release specify the version of the existing docker image (eg. 1.0.12)"
        type: string
        default: "0.0.0"
      target_date:
        description: "In case of \"stable\" release specify the date of the existing docker image in format YYYYMMDD (eg. 20220124)"
        type: string
        default: "20131206"
  repository_dispatch:
    types: [rebuild]
  pull_request:
    types:
      - closed
      - opened

jobs:
  build:
    runs-on: ubuntu-20.04
    strategy:
      fail-fast: false
      matrix:
        include:
          - ros-distro: foxy
            platforms: "linux/amd64, linux/arm64"
          - ros-distro: galactic
            platforms: "linux/amd64, linux/arm64"
          - ros-distro: humble
            platforms: "linux/amd64, linux/arm64"

    steps:

      - name: Checkout
        uses: actions/checkout@v2

      - name: Build Docker Image
        uses: husarion-ci/ros-docker-img-action@v0.1
        with:
          dockerhub_username: ${{ secrets.DOCKERHUB_USERNAME }}
          dockerhub_token:  ${{ secrets.DOCKERHUB_TOKEN }}
          build_type: ${{ inputs.build_type }}
          ros_distro: ${{ matrix.ros-distro }}
          platforms: ${{ matrix.platforms }}
          # variables important only for stable release
          target_distro: ${{ inputs.target_distro }}
          target_release: ${{ inputs.target_release }}
          target_date: ${{ inputs.target_date }}
```

## Inputs

| Name | Required | Default value | Description |
| - | - | - | - |
| `dockerhub_username` | yes | - | DockerHub username |
| `dockerhub_token` | yes | - | DockerHub token |
| `main_branch_name` | no | main | the name of the main branch - typically "main" or "master" |
| `dockerfile` | no | Dockerfile | target Dockerfile |
| `repo_name` | no | - | custom repository name: keep empty to get automatically from repo name |
| `prefix` | no | - | Custom prefix after account/repo_name to be used as the `PREFIX` argument in the `Dockerfile` |
| `build_type` | no | development | "stable" or "development" release |
| `branch_name` | no | main | Optional branch name to be used as the `BRANCH_NAME` argument in the `Dockerfile` |
| `ros_distro` | no | humble | Target ROS distribution to be used as the `ROS_DISTRO` argument in the `Dockerfile` |
| `platforms` | no | linux/amd64, linux/arm64 | target architectures |
| `target_distro` | no | ardent | [Stable release only] Point to the ROS distro (part of tag) of the **EXISTING** docker image on DockerHub |
| `target_release` | no | 0.0.0 | [Stable release only] Point to the release number (part of tag) of the **EXISTING** docker image on DockerHub |
| `target_date` | no | 20131206 | [Stable release only] Point to the date (part of tag) of the **EXISTING** docker image on DockerHub |