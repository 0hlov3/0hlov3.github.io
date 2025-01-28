+++
title = 'Building and Running Multi-Arch Containers with Podman'
subtitle = 'A Guide to Docker Hub Integration'
author = "0hlov3"
date = 2025-01-27T20:01:00-00:00
draft = false
image = "nick-karvounis-SmIM3m8f3Pw-unsplash.jpg"
tags = ['Podman','Multi-Architecture','Containerization','Docker Hub','QEMU']
+++
{{< figure src="nick-karvounis-SmIM3m8f3Pw-unsplash.jpg" caption="Photo by {{< newtablink \"https://unsplash.com/@nickkarvounis?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\" >}}Nick Karvounis{{< /newtablink >}} on {{< newtablink \"https://unsplash.com/photos/white-wooden-house-near-body-of-water-at-daytime-SmIM3m8f3Pw?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash\" >}}Unsplash{{< /newtablink >}}" alt="Container Boat House" >}}

In today’s software ecosystem, containerization has become a cornerstone of modern development and deployment workflows. Tools like {{< newtablink \"https://www.docker.com/\" >}}Docker{{< /newtablink >}} and {{< newtablink \"https://podman.io/\" >}}Podman{{< /newtablink >}} have empowered developers to encapsulate applications and their dependencies into portable, lightweight containers. However, as we move toward a multi-architecture world—driven by the rise of ARM-based systems such as Raspberry Pi, Apple Silicon, and even ARM-based cloud instances—ensuring that your containers can run seamlessly across different architectures is critical.

This article focuses on building and running multi-architecture (multi-arch) containers using Podman, a daemonless, open-source alternative to Docker, designed for secure and efficient container management. We will guide you through the process of addressing common challenges, such as the inability to natively run ARM containers on x86 hosts, by leveraging tools like QEMU and binfmt_misc. By the end of this guide, you'll not only learn how to build and push multi-arch containers to {{< newtablink \"https://hub.docker.com/\" >}}Docker Hub{{< /newtablink >}} but also understand the key configurations needed to ensure smooth execution across architectures.

Whether you're a developer targeting IoT devices, experimenting with Raspberry Pi, or deploying applications in a heterogeneous cloud environment, mastering multi-arch containerization will broaden your deployment capabilities and future-proof your containerized workloads.

## Creating a Personal Access Token for Docker Hub
To push container images to Docker Hub using Podman, you'll need a Personal Access Token, much like you would when using the Docker CLI. Users who rely on Docker Desktop might not have encountered this step before, as Docker Desktop often manages authentication seamlessly in the background. However, when working with Podman or other CLI tools, this step becomes essential. Follow these instructions to generate your token:

1. Navigate to Your Account Settings
   
   Log in to {{< newtablink \"https://hub.docker.com/\" >}}Docker Hub{{< /newtablink >}}, then click on your profile picture (located in the top-right corner) and select Account Settings from the dropdown menu.
2. Access the Personal Access Token Section
   
   In the settings menu, click on Security (or a similar section, depending on UI updates) and locate the Personal Access Tokens option. Select Generate New Token to proceed.
3. Provide a Descriptive Token Name

   When prompted, give your token a meaningful description to easily identify its purpose in the future. For example, use names like "Development Machine 1" or "Podman on Workstation".
4. Set an Expiration Date
   
   Choose an appropriate expiration period for your token. It's best to select 30 or 90 days for better security hygiene. While longer durations or no expiration might seem convenient, they increase the risk of token misuse. For most use cases, a 90-day expiration strikes a good balance between convenience and security.
5. Configure Access Permissions
   
   Ensure the Access Permissions for the token are set to Read & Write. This is necessary for pushing images to Docker Hub.

Once you’ve generated the token, copy it immediately and store it securely. You won’t be able to view the token again after navigating away from the page. Tools like a password manager or secure notes in your terminal (e.g., `pass`) are excellent for safely storing sensitive credentials.

By completing this step, you’re now ready to authenticate Podman with Docker Hub and start pushing your container images.

## Authenticate Podman with Docker Hub
Now that you have your Personal Access Token, it’s time to authenticate Podman with Docker Hub. The authentication process is straightforward and similar to using the Docker CLI. Follow these steps to log in:
1. Open your terminal and run the following command:
   ```bash
   podman login docker.io
   ```
2. When prompted, provide your Docker Hub username.
3. For the password, use the Personal Access Token you generated earlier instead of your Docker Hub account password.

Here’s an example of what the process looks like:
```bash
$ podman login docker.io
Username: your_dockerhub_username
Password: (enter your personal access token)
Login Succeeded!
```

If you see the message "Login Succeeded!", your Podman client is now authenticated with Docker Hub, and you’re ready to push and pull container images.
### Important Note About Credential Storage
By default, Podman stores your registry credentials in an ephemeral directory:
`${XDG_RUNTIME_DIR}/containers/auth.json`, which typically resolves to a subdirectory in `/run`. Because `/run` is a transient location, its contents disappear after a system reboot. This means you’ll need to log in again after every reboot unless you configure a persistent location.
#### Persisting Registry Credentials
To avoid re-authenticating after each reboot, specify a persistent location for your credentials when logging in:
```bash
podman login --authfile $HOME/.config/containers/auth.json docker.io
```
This command saves the authentication token to `$HOME/.config/containers/auth.json`, ensuring it remains intact across reboots.

For more details about how Podman handles authentication, refer to the `containers-auth.json` manual:
```bash
man containers-auth.json
```

### Why Use a Personal Access Token?
Unlike traditional passwords, Personal Access Tokens provide a more secure way to authenticate. They can be scoped to specific permissions (e.g., read/write) and are easier to manage, as they can be revoked or regenerated without affecting your main account credentials.

Remember: Store your token securely, and avoid sharing it or embedding it in scripts without encryption. Use environment variables or secret management tools to handle sensitive credentials when automating workflows.

## Running Containers from Docker Hub
When using Podman to run containers, you may occasionally encounter an error message like this:
```bash
❯ podman run -ti --entrypoint='' library/node:lts-alpine /bin/sh
Error: short-name "library/node:lts-alpine" did not resolve to an alias and no unqualified-search registries are defined in "/etc/containers/registries.conf"
```
This error occurs because Podman does not automatically default to the docker.io registry for unqualified image names. Unlike Docker, Podman uses a more explicit approach to image resolution, requiring either a fully qualified image name or predefined aliases.
### Understanding the Cause
Podman relies on a configuration file, `shortnames.conf`, to map unqualified image names to fully qualified registry paths. This file contains a list of aliases for common image sources. You can inspect these aliases with the following command:
```shell
cat /etc/containers/registries.conf.d/00-shortnames.conf
```
The output will show mappings of short names (e.g., `library/node`) to their corresponding registries (e.g., `docker.io/library/node`). However, if the alias for your image is not defined or the registry is not explicitly specified, Podman will throw the error mentioned above.
### Fixing the Issue
To resolve this, explicitly specify the registry in your Podman command. For example:
```shell
❯ podman run -ti --entrypoint='' docker.io/library/node:lts-alpine /bin/sh
```
By adding `docker.io/` before the image name, you inform Podman to pull the image from Docker Hub. This approach ensures that Podman knows exactly which registry to use, avoiding the ambiguity that leads to errors.
### Best Practices
To prevent similar issues in the future:
1. Use Fully Qualified Image Names:
   
   Always include the registry (e.g., docker.io, quay.io) when pulling or running images.
2. Check the Shortnames Configuration: 
   
   If you frequently use certain images, consider updating or reviewing the `shortnames.conf` file for mappings.
3. Default Registry Configuration: 
   
   You can define default registries in `/etc/containers/registries.conf`, although this is not always recommended for production environments where explicit configurations are preferred.

By following these steps, you can avoid common pitfalls and streamline your Podman workflows.

## Running an ARM Container and Resolving the Error
Now that you can successfully pull and run containers from Docker Hub, let’s try running an ARM64 container on an AMD64 system. This is a common scenario when testing or deploying applications intended for ARM-based devices, such as Raspberry Pi or Apple Silicon.

Run the following command to attempt running an ARM64 container:
```bash
❯ podman run -ti --entrypoint='' --platform linux/arm64/v8 docker.io/library/node:lts-alpine /bin/sh
```
You might see the following error:
```bash
Trying to pull docker.io/library/node:lts-alpine...
Getting image source signatures
Copying blob 6cb00bd14f6e skipped: already exists
Copying blob fab9c87405a8 skipped: already exists
Copying blob 5f0f091c674d skipped: already exists
Copying blob 52f827f72350 skipped: already exists
Copying config cfcefdbe4c done   |
Writing manifest to image destination
{"msg":"exec container process /bin/sh: Exec format error","level":"error","time":"2025-01-27T21:51:00.575423Z"}
```
### Understanding the Error
The "Exec format error" occurs because the binary being executed inside the ARM64 container (`/bin/sh`) is incompatible with the host system’s architecture (AMD64). This happens when the system is not configured to handle non-native architectures.

To fix this issue, you need to enable support for multi-architecture emulation by installing and configuring the required dependencies.

### Installing the Required Dependencies
To run ARM64 containers on an AMD64 system, you need to install the following packages:

1. qemu-user-static

   This package provides the QEMU binaries for static user-mode emulation. It includes emulators for various architectures, such as ARM64 (`qemu-aarch64`), ARM (`qemu-arm`), and more.
   - Purpose: Provides the core emulation binaries.
   - Use Case: Enables the host system (e.g., x86_64) to execute binaries compiled for other architectures (e.g., ARM64).
   - Note: On its own, this package doesn’t configure the system to use QEMU automatically.
2. qemu-user-static-binfmt

   This package complements `qemu-user-static` by setting up binfmt_misc, a kernel feature that allows the system to recognize and use QEMU for non-native binaries automatically.
   - Purpose: Automates the configuration of `binfmt_misc` for static emulation.
   - Use Case: Ensures seamless multi-architecture support for container runtimes like Podman or Docker.
   - Dependency: Installing this package will also install `qemu-user-static`.

On systems like Arch Linux, install these packages using:
```bash
sudo pacman -S qemu-user-static-binfmt qemu-user-static
```

### Configuring Multi-Arch Support
To enable multi-architecture support, you need to configure QEMU on your system. This allows Podman to emulate and run containers built for architectures other than your host's native one. You have two main approaches to set this up:

#### 1. Using the Multiarch QEMU Container
Run the following command to configure QEMU for handling non-native architectures:
```bash
sudo podman run --rm --privileged docker.io/multiarch/qemu-user-static --reset -p yes
```
This command installs and configures QEMU with `binfmt_misc`, a kernel feature that automatically routes foreign architecture binaries through the appropriate QEMU emulator.

#### 2. Enabling the Systemd Binfmt Service
Alternatively, instead of using the QEMU container, you can start the systemd-binfmt.service one time on your system or reboot:
```bash
sudo systemctl start systemd-binfmt.service
```
This service ensures that the required binary format rules are loaded into the kernel, allowing your system to recognize and execute foreign architecture binaries automatically.

### Running the ARM64 Container
With the dependencies installed and QEMU configured, re-run the ARM64 container:
```bash
❯ podman run -ti --entrypoint='' --platform linux/arm64/v8 docker.io/library/node:lts-alpine /bin/sh
```
This time, the container should run successfully:
```bash
Trying to pull docker.io/library/node:lts-alpine...
Getting image source signatures
Copying blob 6cb00bd14f6e skipped: already exists
Copying blob 5f0f091c674d skipped: already exists
Copying blob fab9c87405a8 skipped: already exists
Copying blob 52f827f72350 skipped: already exists
Copying config cfcefdbe4c done   |
Writing manifest to image destination
/ # 
```

You now have a working ARM64 container running on an AMD64 system, thanks to QEMU and Podman’s multi-architecture support.

## Building Multi-Architecture Container Images
Now that you can run an ARM64 container on your system, let’s take the next step: building a multi-architecture container image and pushing it to Docker Hub. This will allow your container to work seamlessly on multiple architectures, such as AMD64 and ARM64.

Below is a simple example to demonstrate the process, along with two approaches for creating and pushing multi-architecture images using Podman.
### Example Dockerfile
To keep things straightforward, let’s use the following Dockerfile. It installs a lightweight utility (`curl`) and prints the system architecture when the container runs:
```Dockerfile
# Use the minimal Alpine base image
FROM --platform=$TARGETPLATFORM alpine:latest

# Set environment variables
ENV PACKAGES="curl"

# Install the desired package
RUN apk add --no-cache $PACKAGES

# Print the architecture and verify the installed package
CMD ["sh", "-c", "echo Running on $(uname -m); curl --version"]
```

- `--platform=$TARGETPLATFORM`: This dynamically adjusts the image build process for the target platform (e.g., AMD64 or ARM64).
- Lightweight Base Image: Using `alpine:latest` ensures the image is minimal and compatible across architectures.
- Verification Command: The `CMD` prints the architecture and verifies the `curl` installation.

### Two Approaches for Building Multi-Arch Images

Podman provides two methods for building and managing multi-architecture images. Choose the one that best fits your workflow.

#### 1. Create Manifest Before Building

This approach creates a manifest list first and then builds the images for multiple architectures in one step.

1. Create a Manifest List:

   A manifest list acts as a container for multiple architecture-specific images:
   ```bash
   podman manifest create docker.io/${DockerHubUsername}/testpodman:0.0.1
   ```
2. Build Multi-Arch Images:
   
   Build images for multiple platforms (linux/amd64 and linux/arm64) and link them to the manifest:
   ```bash
   podman build --platform linux/amd64,linux/arm64 --manifest docker.io/${DockerHubUsername}/testpodman:0.0.1 .
   ```
3. Push the Manifest and Images:
   
   Push the manifest along with the images to Docker Hub:
   ```bash
   podman manifest push docker.io/${DockerHubUsername}/testpodman:0.0.1
   ```

This method is efficient for building and pushing multi-arch images in a single workflow.

#### 2. Adding Images to the Manifest Later
This approach builds each architecture-specific image separately and then adds them to a manifest list.
1. Build Architecture-Specific Images:
   
   Build the images for ARM64 and AMD64 individually:
   ```bash
   podman build --platform linux/arm64 -t docker.io/${DockerHubUsername}/testpodman:arm64 .
   ```
   ```bash
   podman build --platform linux/amd64 -t docker.io/${DockerHubUsername}/testpodman:amd64 .
   ```
2. Create a Manifest List:
   
   Create a new manifest list for the multi-architecture image:
   ```bash
   podman manifest create docker.io/${DockerHubUsername}/testpodman:0.0.2
   ```
3. Add Images to the Manifest:
   
   Link the architecture-specific images to the manifest list:
   ```bash
   podman manifest add docker.io/${DockerHubUsername}/testpodman:0.0.2 docker.io/${DockerHubUsername}/testpodman:amd64
   podman manifest add docker.io/${DockerHubUsername}/testpodman:0.0.2 docker.io/${DockerHubUsername}/testpodman:arm64
   ```
4. Push the Manifest and Images:
   
   Push everything to Docker Hub:
   ```bash
   podman manifest push --all  docker.io/${DockerHubUsername}/testpodman:0.0.2
   ```

This approach provides more flexibility for building and managing architecture-specific images individually.

#### Key Differences Between the Approaches

| Aspect      | Manifest First                          | Manifest Later                              |
|-------------|-----------------------------------------|---------------------------------------------|
| Workflow    | Build and push in one step              | Separate build and manifest creation steps  |
| Flexibility | Less flexible                           | More control over individual images         |
| Use Case    | Efficient for straightforward workflows | Ideal for advanced or custom configurations |

#### Verifying the Multi-Arch Image
After pushing, you can verify the manifest and its architecture-specific images:
```bash
podman manifest inspect docker.io/${DockerHubUsername}/testpodman:0.0.2
```
The output will list the platforms (e.g., `linux/amd64` and `linux/arm64`) included in the manifest.

With these steps, you can build and push multi-architecture container images using Podman. Whether you prefer an all-in-one approach or a more flexible workflow, Podman provides the tools to manage your multi-arch builds effectively.

## Testing our Multi-Arch Images
Once you've built and pushed your multi-architecture images to Docker Hub, it's time to test them on different platforms to ensure they work as expected. Below are the commands and expected outputs for running the ARM64 and AMD64 versions of the image.
### Testing the ARM64 Image
To test the ARM64 version of the image, use the following command:
```bash
podman run --platform linux/arm64 docker.io/${DockerHubUsername}/testpodman:0.0.1
```
Output:
```bash
Trying to pull docker.io/${DockerHubUsername}/testpodman:0.0.1...3/testpodman:0.0.1
Getting image source signatures
Copying blob c59179de3fa6 skipped: already exists  
Copying blob 52f827f72350 skipped: already exists  
Copying config d1affdfbb2 done   | 
Writing manifest to image destination
Running on aarch64
curl 8.11.1 (aarch64-alpine-linux-musl) libcurl/8.11.1 OpenSSL/3.3.2 zlib/1.3.1 brotli/1.1.0 zstd/1.5.6 c-ares/1.34.3 libidn2/2.3.7 libpsl/0.21.5 nghttp2/1.64.0
Release-Date: 2024-12-11
Protocols: dict file ftp ftps gopher gophers http https imap imaps ipfs ipns mqtt pop3 pop3s rtsp smb smbs smtp smtps telnet tftp ws wss
Features: alt-svc AsynchDNS brotli HSTS HTTP2 HTTPS-proxy IDN IPv6 Largefile libz NTLM PSL SSL threadsafe TLS-SRP UnixSockets zstd
```
The output confirms that the image is running on the ARM64 architecture (`aarch64`) and includes the expected `curl` version (`8.11.1`) along with its compiled features and supported protocols.
### Testing the AMD64 Image
Next, test the AMD64 version of the image with the following command:
```bash
podman run --platform linux/amd64 docker.io/${DockerHubUsername}/testpodman:0.0.1
```
Output:
```bash
Trying to pull docker.io/${DockerHubUsername}/testpodman:0.0.1...3/testpodman:0.0.1
Getting image source signatures
Copying blob b2a18c480def done   | 
Copying blob 1f3e46996e29 skipped: already exists  
Copying config ddc1406d47 done   | 
Writing manifest to image destination
Running on x86_64
curl 8.11.1 (x86_64-alpine-linux-musl) libcurl/8.11.1 OpenSSL/3.3.2 zlib/1.3.1 brotli/1.1.0 zstd/1.5.6 c-ares/1.34.3 libidn2/2.3.7 libpsl/0.21.5 nghttp2/1.64.0
Release-Date: 2024-12-11
Protocols: dict file ftp ftps gopher gophers http https imap imaps ipfs ipns mqtt pop3 pop3s rtsp smb smbs smtp smtps telnet tftp ws wss
Features: alt-svc AsynchDNS brotli HSTS HTTP2 HTTPS-proxy IDN IPv6 Largefile libz NTLM PSL SSL threadsafe TLS-SRP UnixSockets zstd
```
The output confirms the image is running on the AMD64 architecture (`x86_64`) and, like the ARM64 version, verifies that `curl` is installed and functional.

### Key Takeaways

1. Architecture-Specific Execution:
   
   The outputs demonstrate that the same container image works seamlessly on both ARM64 and AMD64 architectures. This validates the multi-architecture build process.
2. Environment-Specific Testing: 
   
   You can test your multi-arch images on physical devices, virtual machines, or emulated environments using QEMU.
3. Feature Consistency: 
   
   Regardless of the platform, the image includes the same dependencies (like `curl`) and functionality, ensuring consistent behavior across architectures.

Testing your multi-arch images is a crucial step to verify that your builds are platform-compatible, providing confidence for production deployments or sharing with the community.

## Conclusion
Building and managing multi-architecture container images has become a critical skill in today’s world of diverse computing environments. Whether you're targeting x86 systems, ARM-based devices like Raspberry Pi, or cloud environments with heterogeneous architectures, tools like Podman provide the flexibility and power to make multi-arch containerization straightforward and efficient.

In this article, we walked through the entire process—from enabling multi-architecture support on your system to creating and pushing multi-arch images to Docker Hub. We explored two approaches for managing manifests and showcased a simple yet effective example with an Alpine-based image.

The key takeaway is that Podman not only offers a robust alternative to Docker but also excels in multi-platform workflows, especially when paired with tools like QEMU for emulation. By leveraging these capabilities, you can future-proof your containerized applications, ensuring compatibility and portability across architectures.

Now that you’ve mastered the basics, you’re ready to experiment with more complex workflows, automate multi-arch builds in CI/CD pipelines, or optimize your containers for performance on specific architectures. The possibilities are endless, and with Podman’s growing ecosystem, you’re well-equipped to tackle the challenges of modern containerization.

## Links
- {{< newtablink \"https://github.com/multiarch/qemu-user-static\" >}}qemu-user-static{{< /newtablink >}}
- {{< newtablink \"https://github.com/containers/podman/discussions/12899\" >}}Podman Discussions: Multi-Arch Builds{{< /newtablink >}}
- {{< newtablink \"https://dev.to/tkral/podman-build-amd64-images-on-m1-294g\" >}}Podman Build AMD64 Images on M1{{< /newtablink >}}
- {{< newtablink \"https://github.com/nodejs/docker-node/issues/2074\" >}}Node.js Docker Image Issue #2074{{< /newtablink >}}
- {{< newtablink \"https://pnpm.io/installation#using-corepack\" >}}PNPM Installation Using Corepack{{< /newtablink >}}
- {{< newtablink \"https://hub.docker.com/_/node/tags?name=lts-alpine\" >}}Node LTS Alpine Tags on Docker Hub{{< /newtablink >}}
- {{< newtablink \"https://wiki.archlinux.org/title/Podman#Foreign_architectures\" >}}Arch Wiki - Foreign architectures{{< /newtablink >}}

## Don‘t trust me

The author is not responsible for any errors or damages resulting from the use of this information.

If you have any questions or suggestions for improvement, please feel free to reach out.