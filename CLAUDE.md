# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Docker Machine driver for Triton (Joyent's container-native infrastructure). It allows users to create and manage Docker hosts on Triton cloud infrastructure through the `docker-machine` CLI tool.

## Build and Development Commands

### Building the Driver
```bash
go build -o docker-machine-driver-triton
```

### Installing from Source
```bash
go get -u github.com/TritonDataCenter/docker-machine-driver-triton
export PATH=$PATH:$GOPATH/bin
```

### Testing
No specific test framework is configured in this project. Use standard Go testing practices:
```bash
go test ./...
```

## Architecture

### Core Components

**Main Entry Point** (`main.go:8`): Simple plugin registration that registers the Driver with Docker Machine's plugin system.

**Driver Implementation** (`driver.go`): Contains the main `Driver` struct that implements the Docker Machine driver interface with these key responsibilities:

- **Authentication**: Supports multiple auth methods (SSH keys, SSH agent, base64-encoded key material)
- **Instance Management**: Creates, starts, stops, restarts, and removes Triton instances
- **Configuration**: Handles all driver-specific flags and environment variables
- **State Management**: Maps Triton instance states to Docker Machine states

### Key Driver Methods

- `Create()` (`driver.go:335`): Provisions new Triton instances with support for tags and metadata
- `GetState()` (`driver.go:594`): Maps Triton states (running, stopped, provisioning, etc.) to Docker Machine states
- `client()` (`driver.go:204`): Creates authenticated Triton API client with flexible auth options
- `PreCreateCheck()` (`driver.go:440`): Validates images and packages before instance creation

### Configuration Flags

The driver accepts configuration via CLI flags or environment variables (with `SDC_` prefix for historical reasons):

**Required:**
- `--triton-account` / `SDC_ACCOUNT`: Triton account username
- `--triton-key-id` / `SDC_KEY_ID`: SSH key fingerprint  
- `--triton-url` / `SDC_URL`: CloudAPI endpoint URL

**Authentication Options:**
- `--triton-key-path` / `SDC_KEY_PATH`: Path to SSH private key
- `--triton-key-material`: Base64-encoded SSH private key content (for Rancher integration)

**Instance Options:**
- `--triton-image`: VM image (defaults to "debian-8")
- `--triton-package`: Instance size (defaults to "k4-highcpu-kvm-250M")
- `--triton-tags` / `SDC_TAGS`: Comma-separated key=value tags
- `--triton-mdata` / `SDC_MDATA`: Comma-separated key=value metadata

### Dependencies

- `github.com/docker/machine`: Docker Machine SDK
- `github.com/TritonDataCenter/triton-go`: Official Triton Go SDK for API interactions
- Go 1.19+

## Usage Examples

See README.md for complete usage examples. The driver integrates with `docker-machine create -d triton` commands.