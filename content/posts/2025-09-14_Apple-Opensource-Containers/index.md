+++
title = "Apple’s New container"
subtitle = "Open Source Containers on macOS"
date = 2025-09-14T20:00:00
image = "github-container-screenshot.png"
tags = ["Apple", "Containers", "macOS", "Podman", "Docker"]
+++

![Terminal session showing Apple’s container CLI in use. The service is started with container system start, followed by container ls showing no containers. A Python container is run to serve a simple ‘Hello, world!’ page. container ls then shows the running container with an assigned IP address 192.168.64.2. Finally, curl http://192.168.64.2 outputs ‘Hello, world!’.](./github-container-screenshot.png)

When you think of containers, you probably think about Docker or Podman. That was true for me too — until I first heard about Apple’s own container tool at Container Days.

Apple has quietly released an open-source project called container. Unlike Docker Desktop, it runs natively on macOS, powered by Apple’s Containerization and Virtualization frameworks.

> A tool for creating and running Linux containers using lightweight virtual machines on a Mac. It is written in Swift, and optimized for Apple silicon. 

In this post, we’ll explore the installation process, CLI differences compared to Docker and Podman, building multi-arch images, running containers, and finally wrap up with some conclusions.

## Installation

Before you begin, make sure you have:
- A Mac with Apple Silicon (M1/M2/M3).
- macOS 15.5 or later (best support on macOS 16 “Tahoe”).

To install container, head over to the {{< newtablink "https://github.com/apple/container" >}}apple/container{{< /newtablink >}} GitHub page and download the latest {{< newtablink "https://github.com/apple/container/releases" >}}Release{{< /newtablink >}} Installer.

> TIP: Choose the signed installer, it’s notarized by Apple and ensures the binary hasn’t been tampered with. This avoids the extra hassle of disabling Gatekeeper or manually approving unsigned binaries in System Settings. It also means you get automatic code-signing verification at install time, which is a good security practice when running a system-level service.

```bash
# Example: install the signed package
export CONTAINER_VERSION="0.4.1"
curl -LO https://github.com/apple/container/releases/download/${CONTAINER_VERSION}/container-${CONTAINER_VERSION}-installer-signed.pkg
sudo installer -pkg container-${CONTAINER_VERSION}-installer-signed.pkg -target /
# Start the system service
container system start
# Verify that the container system is healthy
container system status
# Verify installation
container --version
```

With that, you’re ready to pull and run your first container.

## CLI vs Docker / Podman
Apple’s container CLI looks familiar if you’ve used Docker or Podman, but the execution model is very different:
| Feature     | Apple `container`                                 | Docker / Podman                                                |
| ----------- | ------------------------------------------------- | -------------------------------------------------------------- |
| Isolation   | Each container runs in its **own lightweight VM** | Containers share a host kernel (or a shared Linux VM on macOS) |
| Performance | Optimized for Apple Silicon, faster cold starts   | More overhead on macOS due to Docker Desktop VM                |
| Networking  | Each container gets its own IP                    | Often NAT + port mappings                                      |
| Ecosystem   | New, limited features                             | Mature, rich ecosystem (Compose, Swarm, plugins, etc.)         |

So while many familiar commands exist (`container run`, `container pull`, `container images`, `container push`), expect some differences when moving complex Docker workflows.

One small but notable difference: there’s no `container ps`. Instead, the equivalent is `container ls` (for listing running containers) or `container ls -a` (for all containers). It’s a tiny change, but it tripped me up more than once.

## Running Containers
Running is straightforward:
```bash
# run interactively
container run -it alpine sh
# mount a local directory
container run -it -v $(pwd):/app -w /app alpine sh
# list running containers
container ls
container ls -a
# list images
container images ls
```
Each container gets its own micro-VM and dedicated IP, which changes networking behavior compared to Docker. You can read more about networking in the {{< newtablink "https://github.com/apple/container/blob/main/docs/how-to.md#create-and-use-a-separate-isolated-network" >}}Documentation{{< /newtablink >}}.

## Registries
container can interact with standard OCI-compliant registries like Docker Hub, GitHub Container Registry, or a private Harbor instance. Logging in works much like Docker or Podman:
```bash
# Example: login to Docker Hub
container registry login --username ${REGISTRY_USERNAME} docker.io
```
When you run this command, you’ll be prompted for a password.

- For Docker Hub and most other registries, it’s best practice to use a Personal Access Token (PAT) instead of your account password.
- Always set an expiration date for your PAT, this limits the damage in case it gets leaked.

Once logged in, you can container push and container pull images from that registry just like you would with Docker.

## Building Containers
On Apple Silicon, images are built for ARM64 by default. Let’s start with a simple example.

### Step 1: Create a Dockerfile
```dockerfile
# Dockerfile
FROM alpine:latest
CMD ["uname", "-m"]
```
### Step 2: Build the Image
```bash
container build -t docker.io/${REGISTRY_USERNAME}/apple-container-test .
```
### Step 3: Inspect the Image
```bash
container image ls -v
```
**Example output:**
```bash
NAME                         TAG     INDEX DIGEST  OS     ARCH   VARIANT  SIZE     CREATED               MANIFEST DIGEST
${REGISTRY_USERNAME}/apple-container-test  0.0.1   REDACTED      linux  arm64           4,1 MB   2025-07-15T11:01:16Z  REDACTED
${REGISTRY_USERNAME}/apple-container-test  latest  REDACTED      linux  arm64           4,1 MB   2025-07-15T11:01:16Z  REDACTED
```
### Step 4: Tag & Push
```bash
# Tag the image with a version
container image tag docker.io/${REGISTRY_USERNAME}/apple-container-test:latest \
  docker.io/${REGISTRY_USERNAME}/apple-container-test:0.0.1

# Push to Docker Hub
container image push docker.io/${REGISTRY_USERNAME}/apple-container-test:0.0.1
container image push docker.io/${REGISTRY_USERNAME}/apple-container-test:latest
```
## Building Multi-Arch Images
The real power comes from building multi-architecture images that support both amd64 and arm64.
Building a Multi Arch image (default on Apple Silicon)
```bash
# Build for both amd64 and arm64
container build --arch amd64,arm64 \
  --tag docker.io/${REGISTRY_USERNAME}/apple-container-test:0.0.2 .

# Update the "latest" tag
container image tag docker.io/${REGISTRY_USERNAME}/apple-container-test:0.0.2 \
  docker.io/${REGISTRY_USERNAME}/apple-container-test:latest

# Push both
container image push docker.io/${REGISTRY_USERNAME}/apple-container-test:0.0.2
container image push docker.io/${REGISTRY_USERNAME}/apple-container-test:latest

```
When pushed, your registry (e.g., Docker Hub) will show multiple digests under the same tag, one per architecture. For example:
- linux/amd64 digest
- linux/arm64 digest

This way, the right image will be pulled automatically depending on the host platform.

![Docker Hub repository view showing container image tags 0.0.1, 0.0.2, and latest. Tag 0.0.1 has an ARM64 image, while tag 0.0.2 includes both AMD64 and ARM64 images. The latest tag points to the ARM64 image. Each entry shows digest, OS/ARCH, and last pull information.](./dockerhub-container-build.png)

## Conclusion
Apple’s container is a fresh take on running containers natively on macOS:

- Lightweight, no Docker Desktop VM overhead
- Stronger isolation, one VM per container
- Optimized performance, designed for Apple Silicon

That said, it’s still early days: some advanced features like Docker Compose, richer multi-arch workflows, and broader ecosystem compatibility are not yet there.

If you’re developing on macOS with Apple Silicon and mostly need single-container workflows, this tool is absolutely worth trying. It may not replace Docker or Podman for every use case today, but it’s an exciting step toward first-class, open-source container support on macOS.

Give it a try, and if you run into issues, consider opening an issue on GitHub. With community feedback and contributions, container has the potential to become a great native alternative for container development on macOS.

# Sources & Further Reading

- {{< newtablink "https://github.com/apple/container/blob/main/docs/how-to.md" >}}container/docs/how-to.md{{< /newtablink >}}
- {{< newtablink "https://github.com/apple/container/blob/main/docs/technical-overview.md#technical-overview" >}}container/docs/technical-overview.md{{< /newtablink >}}
- {{< newtablink "https://apple.github.io/container/documentation" >}}container Documentation{{< /newtablink >}}
- {{< newtablink "https://blog.schoenwald.aero/where-to-start-with-docker-part-1-6d3000777018" >}}Where to start with Docker — Part 1{{< /newtablink >}}

## Don’t Trust Me - Seriously

The author takes no responsibility for any mishaps, broken servers, or existential crises caused by following this information.

Found a mistake? Open an issue or PR on GitHub, or ping me on Mastodon/LinkedIn/Twitter. Let’s improve it together.

Also, this isn’t an ad - unless my enthusiasm and advocacy for cool stuff count as advertising.
