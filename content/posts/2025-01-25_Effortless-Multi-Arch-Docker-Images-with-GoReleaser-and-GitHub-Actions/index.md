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
- Podman or Docker: A containerization tool for building and running multi-arch Docker images.
  - We focus on Docker, cause building with podman is exclusively available with `GoReleaser Pro`.

### Supported Platforms
This guide focuses on GitHub Actions for CI/CD, but GoReleaser also supports GitLab, Gitea, and Bitbucket. Consult {{< newtablink \"https://goreleaser.com/ci/\" >}}GoReleaser’s documentation{{< /newtablink >}} for more details.

### Naming Conventions
GitHub repository names must be lowercase since GoReleaser does not support uppercase letters. Choose a compliant name to avoid issues.

## Setting Up the Repository
To get started, create a new Git repository for your project. If you haven't already, you can do this either locally or directly on GitHub.

If you haven't created the repository on GitHub yet, you should create one and follow these next steps:

1. Create a new repository on GitHub and name it (e.g., goreleaser-multi-arch-docker).
2. Clone the repository locally to work on it:
```bash
git clone https://github.com/<your-github-username>/goreleaser-multi-arch-docker.git
cd goreleaser-multi-arch-docker
```
If you prefer to initialize a new repository locally first and then push it to GitHub, use the following commands:
```bash
mkdir goreleaser-multi-arch-docker
cd goreleaser-multi-arch-docker
git init
```
After initializing, configure your repository:
```bash
git remote add origin https://github.com/<your-github-username>/goreleaser-multi-arch-docker.git
git branch -M main
```
Make sure to replace `<your-github-username>` with your actual GitHub username.

Now your repository is ready for development, and you can start adding files for your Go project and GoReleaser configuration.

## Creating a Dockerfile
To containerize our Go application, we need a minimal and secure Dockerfile. Since GoReleaser handles the compilation and packaging of our application, the Dockerfile only needs to include the final executable.

We'll use Google's distroless static image, which provides a small, secure, and efficient base image, reducing the attack surface and unnecessary dependencies. However, it's crucial to ensure that the chosen base image supports all the architectures we plan to build later.
```Dockerfile
# Use a minimal, secure, and non-root base image
FROM gcr.io/distroless/static:nonroot

# Copy the compiled binary into the container
COPY goreleaser-multi-arch-docker /helloworld

# Set the entrypoint to execute the application
ENTRYPOINT [ "/helloworld" ]
```

## Writing the Go Application
Now, let's create a simple Go application that prints system details, including the operating system architecture, and the current user. This helps us verify that the multi-architecture builds are working correctly when we run the container on different platforms.
### Creating the `helloworld` Application
Inside your project directory, create a new folder for the Go application:
```bash
mkdir -p helloworld
```
Then, create the `main.go` file:
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
#### Explanation:
- user.Current(): Fetches the current user’s details.
- Error handling: we don't do error handling here.
- runtime.GOOS and runtime.GOARCH: Prints the operating system and CPU architecture, helping verify multi-arch builds.
### Compiling the Application
Since we aim for a statically linked binary to ensure compatibility across different environments, compile the Go application using:
```bash
go build -ldflags="-s -w" -o bin/helloworld helloworld/main.go
```
- The `-ldflags="-s -w"` flags reduce binary size by stripping debugging information.
- The `-o` flag specifies the output binary location.
### Running the Application Locally
To test the application before containerizing it, run:
```bash
./bin/helloworld
```
Expected output (on an `amd64` Linux system):
```plaintext
Hello, your-username!
linux amd64

```
If running on an `arm64` system (e.g., Raspberry Pi), the output might be:
```bash
Hello, your-username!
linux arm64
```
This output confirms that the Go binary correctly detects the system architecture, which is crucial when deploying to multi-arch environments.

## Configuring GoReleaser
GoReleaser automates the process of building, packaging, and releasing Go applications. In this section, we’ll configure it to:

- Build multi-architecture binaries for Linux, Windows, and macOS.
- Package the binaries as compressed archives (.zip).
- Build and publish multi-architecture (`linux/amd64`,`linux/arm64`) Docker images using docker buildx.
- Generate a Docker manifest to ensure a single multi-platform image tag.
- Create checksums for release artifacts.
- Generate a changelog from commit messages.

### Creating the Configuration File

Create a `.goreleaser.yaml` file in your project root:
```bash
touch .goreleaser.yaml
```
Now, add the following configuration to the `.goreleaser.yaml`:
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
#### Understanding the Configuration
1. Build Multi-Architecture Binaries
   - Supports `linux`, `windows`, and `darwin (macOS)`.
   - Architectures: `amd64`, `arm`, and `arm64`.
   - CGO disabled (`CGO_ENABLED=0`) to ensure static linking for portability.
2. Package Artifacts
   - Archives are stored in .zip format, named as:
     ```plaintext
     goreleaser-multi-arch-docker_v1.0.0_linux_amd64.zip
     ```
3. Multi-Arch Docker Builds
   - Uses `docker buildx` to build images for `amd6`4 and `arm64`.
   - Adds Open Container Initiative (OCI) labels for better image metadata.
4. Docker Manifests
   - Combines architecture-specific images into a single multi-arch image.
   - Example published tags:
     ```bash
     ghcr.io/your-username/goreleaser-multi-arch-docker:v1.0.0
     ghcr.io/your-username/goreleaser-multi-arch-docker:latest
     ```
5. Checksum & Changelog Generation
   - A checksum file (`checksums.txt`) is created for verification.
   - A changelog is generated from commit messages, excluding non-relevant ones (`docs`, `test`, `ci`).
#### Running GoReleaser
Before we can release our application, we need to test the GoReleaser setup locally and later trigger the automated release process through GitHub Actions.
##### Running GoReleaser Locally
To ensure everything is correctly configured before pushing a release, we can run GoReleaser in snapshot mode. This will perform a dry-run without actually publishing any artifacts.

First, set your GitHub username as an environment variable:
```bash
export REPO_OWNER=<your-github-username>
```
Then, execute GoReleaser in snapshot mode:
```bash
goreleaser release --snapshot --clean
```
What These Flags Do:
- --snapshot: Prevents publishing artifacts (useful for testing).
- --clean: Removes old builds before starting a new one.

## Setting Up GitHub Actions
To fully automate our GoReleaser-based release pipeline, we need a GitHub Actions workflow that will:

- Run GoReleaser when a new Git tag is pushed.
- Build and publish multi-arch Docker images (amd64, arm64).
- Push Docker images to GitHub Container Registry (GHCR).
- Generate GitHub Release artifacts, including compiled binaries.
- Cache Go dependencies to speed up build

### Creating the Workflow File
Create a new workflow file in your repository:
```bash
mkdir -p .github/workflows
touch .github/workflows/release.yaml
```
Then, add the following configuration: `.github/workflows/release.yaml`:
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
### Understanding the Workflow
1. Triggers on Git Tags
   - This workflow only runs when a Git tag (`v*`) is pushed, ensuring that releases follow semantic versioning (e.g., `v1.0.0`).
2. Multi-Architecture Docker Build Setup
   - `docker/setup-qemu-action@v3`: Enables multi-architecture builds for `arm64` and `amd64`.
   - `docker/setup-buildx-action@v3`: Uses BuildKit for faster, more efficient Docker builds.
3. Authentication for GitHub Container Registry (GHCR)
   - Uses `docker/login-action@v3` to log in with the `GITHUB_TOKEN`, allowing GoReleaser to push images.
4. Version-Aware Checkout
   - `fetch-depth: 0` ensures that GoReleaser correctly detects the Git history, which is needed for versioning and changelog generation.
5. Go Setup & Caching
   - Sets up Go using `actions/setup-go@v5`.
   - Caches Go dependencies (`go mod tidy`) to speed up builds in future runs
6. Running GoReleaser
   - Uses the official GoReleaser action (`goreleaser/goreleaser-action@v6`).
   - Runs `goreleaser release --clean` to ensure a fresh build.
### Testing the Workflow
Once the workflow is committed, test it by tagging a release:
```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```
This will trigger the GitHub Actions workflow, building and publishing:
- Multi-arch Docker images (amd64, arm64).
- GitHub release assets (Go binaries).
- Checksums and changelogs for verification.

## Example Repository

For a complete working example, check out my repository: {{< newtablink \"https://github.com/0hlov3/goreleaser-multi-arch-docker\" >}}GoReleaser Multi-Arch Docker{{< /newtablink >}}. This repository contains all the necessary configurations and code to get started with GoReleaser and GitHub Actions.


## Conclusion

With GoReleaser and GitHub Actions, you can automate building and publishing multi-architecture Docker images with minimal configuration. This setup streamlines the release process and ensures your applications are available across different platforms effortlessly.

Try it out, and let me know your thoughts or any improvements you’ve made in the process!

## Don’t Trust Me — Seriously

The author takes no responsibility for any mishaps, broken servers, or existential crises caused by following this information.

If you spot a mistake, have a better way of doing things, or just want to chat about tech, feel free to reach out.

Also, this isn’t an ad — unless my enthusiasm and advocacy for cool stuff count as advertising.