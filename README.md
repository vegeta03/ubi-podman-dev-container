# UBI 8 Podman Development Container for GitHub Codespaces

This repository contains a development container configuration for GitHub Codespaces, featuring UBI 8 (Universal Base Image) with Podman 5.2.2. This setup provides a containerized environment for developing and testing container-based applications using Podman.

## Features

- UBI 8 base image
- Podman 5.2.2
- Rootless Podman configuration
- VS Code integration with useful extensions

## Prerequisites

- GitHub account with Codespaces access
- Basic understanding of containers and Podman

## Getting Started

1. Open this repository in GitHub Codespaces.
2. Wait for the container to build and start.
3. Once the environment is ready, you can start using Podman commands in the terminal.

## Container Configuration

### Dockerfile

The Dockerfile sets up the UBI 8 environment with Podman installed. Key aspects include:

- Multi-stage build for optimized image size
- Installation of Podman 5.2.2 from the COPR repository
- Configuration of rootless Podman
- Setting up necessary dependencies and permissions

### devcontainer.json

The devcontainer.json file configures the GitHub Codespace environment. Notable configurations:

- Privileged container mode for full Podman functionality
- Mounting of user's SSH directory for authentication
- VS Code settings and extensions for container development
- Environment variables for Podman configuration

## Usage

Once the Codespace is running, you can use Podman commands directly in the terminal. For example:

```bash
podman run hello-world
podman build -t myapp .
podman run -it myapp
```
