+++
title = 'Effortless Multi-Arch Docker Images with GoReleaser and GitHub Actions'
subtitle = 'Streamline Your CI/CD Pipeline for Multi-Platform Docker Image Builds'
date = 2025-01-25T20:01:00-00:00
draft = false
image = "dominic-kurniawan-suryaputra-Y3uFSzzw7Wk-unsplash.jpg"
tags = ['GoReleaser','GitHub Actions','Docker','Multi-Arch Containers','CI/CD Automation']
+++
{{< figure src="dominic-kurniawan-suryaputra-Y3uFSzzw7Wk-unsplash.jpg" caption="Photo by {{< newtablink \"https://unsplash.com/@d_ks11?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\" >}}Dominic Kurniawan Suryaputra{{< /newtablink >}} on {{< newtablink \"https://unsplash.com/photos/a-statue-of-a-rocket-sitting-on-top-of-a-rock-Y3uFSzzw7Wk?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\" >}}Unsplash{{< /newtablink >}}" alt="Rocket at Marks Park, Fletcher Street, Tamarama NSW, Australia" >}}

Releasing software can be a tedious process, especially when managing multiple architectures and platforms. GoReleaser is a powerful automation tool for Go projects that simplifies the build, release, and publishing steps, offering extensive customization. With GoReleaser, you can automate everything from compiling binaries and building Docker images to publishing releases with minimal configuration.

In this post, I’ll walk you through how to leverage GoReleaser to build multi-architecture Docker images using GitHub Actions, making your release process streamlined and scalable across diverse environments.

## Prerequisites
Before we dive into the setup, ensure you have the necessary tools installed:

### Required Tools
To follow along, make sure the following tools are installed on your system:
- Git: Used for version control and tagging releases.
- GoReleaser: Automates the release process for Go projects.
- Podman or Docker: A containerization tool for building and running multi-arch Docker images. (we use docker here)

### Supported Platforms
This guide focuses on GitHub Actions for CI/CD, but GoReleaser also supports GitLab, Gitea, and Bitbucket. Consult {{< newtablink \"https://goreleaser.com/ci/\" >}}GoReleaser’s documentation{{< /newtablink >}} for more details.

### Naming Conventions
GitHub repository names must be lowercase since GoReleaser does not support uppercase letters. Choose a compliant name to avoid issues.

## Setting Up the Repository
Initialize a new Git repository:
```bash
mkdir goreleaser-multi-arch-docker
cd goreleaser-multi-arch-docker
git init
```

## Creating a Dockerfile
We need a minimal Dockerfile to build our container image. GoReleaser will copy the compiled binary into the container:

```Dockerfile
FROM gcr.io/distroless/static:nonroot

COPY goreleaser-multi-arch-docker /helloworld

ENTRYPOINT [ "/helloworld" ]
```

## Writing the Go Application
Create a simple Go program that prints system details:
```go
// helloworld/main.go
package main

import (
	"fmt"
	"os/user"
	"runtime"
)

func main() {
	user, _ := user.Current()
	fmt.Printf("Hello %s\n", user.Name)
	fmt.Println(runtime.GOOS, runtime.GOARCH)
}
```
## Configuring GoReleaser
Create `.goreleaser.yaml` in the project root:
```yaml
# https://goreleaser.com
version: 2

project_name: goreleaser-multi-arch-docker

before:
  # https://goreleaser.com/customization/hooks/
  hooks:
    # tidy up and lint
    - go mod tidy
    - go fmt ./...

builds:
  # https://goreleaser.com/customization/build/
  - id: goreleaser-multi-arch-docker
    main: ./helloworld
    goos:
      - linux
      - windows
      - darwin
    goarch:
      - amd64
      - arm
      - arm64
    env:
      - CGO_ENABLED=0
    ldflags:
      - "-s -w"
    mod_timestamp: "{{ .CommitTimestamp }}"

archives:
  - formats: [ 'zip' ]
    name_template: "{{ .ProjectName }}_{{ .Version }}_{{ .Os }}_{{ .Arch }}"

dockers:
  # https://goreleaser.com/customization/docker/
  - use: buildx
    goos: linux
    goarch: amd64
    image_templates:
      - "ghcr.io/{{ .Env.REPO_OWNER }}/{{ .ProjectName }}:{{ .Version }}-amd64"
      - "ghcr.io/{{ .Env.REPO_OWNER }}/{{ .ProjectName }}:latest-amd64"
    build_flag_templates:
      - "--pull"
      - "--platform=linux/amd64"
      - "--label=org.opencontainers.image.created={{.Date}}"
      - "--label=org.opencontainers.image.title={{.ProjectName}}"
      - "--label=org.opencontainers.image.revision={{.FullCommit}}"
      - "--label=org.opencontainers.image.version={{.Version}}"
  - use: buildx
    goos: linux
    goarch: arm64
    image_templates:
      - "ghcr.io/{{ .Env.REPO_OWNER }}/{{ .ProjectName }}:{{ .Version }}-arm64"
      - "ghcr.io/{{ .Env.REPO_OWNER }}/{{ .ProjectName }}:latest-arm64"
    build_flag_templates:
      - "--pull"
      - "--platform=linux/arm64/v8"
      - "--label=org.opencontainers.image.created={{.Date}}"
      - "--label=org.opencontainers.image.title={{.ProjectName}}"
      - "--label=org.opencontainers.image.revision={{.FullCommit}}"
      - "--label=org.opencontainers.image.version={{.Version}}"

docker_manifests:
  # https://goreleaser.com/customization/docker_manifest/
  - name_template: "ghcr.io/{{ .Env.REPO_OWNER }}/{{ .ProjectName }}:{{ .Version }}"
    image_templates:
      - "ghcr.io/{{ .Env.REPO_OWNER }}/{{ .ProjectName }}:{{ .Version }}-amd64"
      - "ghcr.io/{{ .Env.REPO_OWNER }}/{{ .ProjectName }}:{{ .Version }}-arm64"
  - name_template: "ghcr.io/{{ .Env.REPO_OWNER }}/{{ .ProjectName }}:latest"
    image_templates:
      - "ghcr.io/{{ .Env.REPO_OWNER }}/{{ .ProjectName }}:latest-amd64"
      - "ghcr.io/{{ .Env.REPO_OWNER }}/{{ .ProjectName }}:latest-arm64"

checksum:
  name_template: "checksums.txt"

changelog:
  sort: "asc"
  filters:
    exclude:
      - "^docs:"
      - "^test:"
      - "^ci:"
```
## Setting Up GitHub Actions
Create `.github/workflows/release.yaml`:
```yaml
name: goreleaser

on:
  #pull_request:
  #  # Run on PRs for testing
  #  branches:
  #    - main
  push:
    # Run only on tags
    tags:
      - "v*"

permissions:
  contents: write
  packages: write
  issues: write
  id-token: write

jobs:
  #test:
  #  name: Test Pull Request
  #  if: github.event_name == 'pull_request'
  #  runs-on: ubuntu-latest
  #  steps:
  #    - name: Checkout Code
  #      uses: actions/checkout@v4
  #    - name: Set up Go
  #      uses: actions/setup-go@v5
  #      with:
  #        go-version: stable
  #    - name: Install Dependencies
  #      run: go mod tidy
  #    - name: Run Tests
  #      run: go test ./... -v

  release:
    name: GoReleaser
    if: github.event_name != 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: docker/setup-qemu-action@v3
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.repository_owner }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Checkout Code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Set up Go
        uses: actions/setup-go@v5
        with:
          go-version: stable
      - name: Cache Go Modules
        uses: actions/cache@v4
        with:
          path: |
            ~/.cache/go-build
            ~/go/pkg/mod
          key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
          restore-keys: |
            ${{ runner.os }}-go-
      - name: Run GoReleaser
        uses: goreleaser/goreleaser-action@v6
        with:
          distribution: goreleaser
          version: "~> v2"
          args: release --clean
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          DOCKER_USERNAME: ${{ github.actor }}
          DOCKER_PASSWORD: ${{ secrets.GITHUB_TOKEN }}
          REPO_OWNER: ${{ github.repository_owner }}
```

## Example Repository

For a complete working example, check out my repository: {{< newtablink \"https://github.com/0hlov3/goreleaser-multi-arch-docker\" >}}GoReleaser Multi-Arch Docker{{< /newtablink >}}. This repository contains all the necessary configurations and code to get started with GoReleaser and GitHub Actions.


## Conclusion

With GoReleaser and GitHub Actions, you can automate building and publishing multi-architecture Docker images with minimal configuration. This setup streamlines the release process and ensures your applications are available across different platforms effortlessly.

Try it out, and let me know your thoughts or any improvements you’ve made in the process!

## Don‘t trust me

The author is not responsible for any errors or damages resulting from the use of this information.

If you have any questions or suggestions for improvement, please feel free to reach out.