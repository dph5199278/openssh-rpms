# elssh Build Guide – Docker-based RPM Packaging

This document explains how to build **elssh** RPM packages for various Enterprise Linux versions using Docker.

You only need to build the versions you actually require. There is no need to run all commands.

All built RPM packages will be automatically placed in the `./output/` directory on your host machine.

## Prerequisites

- Docker (version 20+ recommended)
- Git
- Sufficient disk space (~10 GB+ recommended)
- Internet connection

## Step 1: Download Sources

You must download the source code and tarballs before building:

```bash
# Download all required sources
env ALL=1 ./pullsrc.sh
```

> **Note**: Run this command only once before starting any builds. It prepares all necessary files for every supported platform.

## Step 2: Building RPMs for Specific Platforms

Choose only the platforms you need and run the corresponding commands.

### Builds

#### Placeholder
```bash
# linux/386   :  x86 [only el5\el6\el7]
# linux/amd64 :  x86_64
# linux/arm64 :  aarch64 (ARM64) [above el6]
# one of them 
BUTILD_PLATFORM=linux/amd64
# 5: el5
# 6: el6
# 7: el7
# 8: el8
# 9: el9
BUILD_VERSION_NUM=5

BUILD_MIRROR=0

BUILD_TAG=elssh:el
```

#### For less than EL9 (CentOS Stream 9 / RHEL 9 / Rocky 9 / AlmaLinux 9)

```bash
env DOCKERBUILD=1 ./pullsrc.sh
# Include
# EL5 (CentOS 5)
# EL6 (CentOS 6)
# EL7 (CentOS 7)
# EL8 (CentOS 8 / RHEL 8 / Rocky 8 / AlmaLinux 8)
# Build Docker image
docker build -t $BUILD_TAG --platform $BUTILD_PLATFORM -f ./docker/Dockerfile.centos --build-arg VERSION_NUM=$BUILD_VERSION_NUM --build-arg MIRROR=$BUILD_MIRROR .

# Build packages
docker run --rm -v .:/data $BUILD_TAG
```

#### For greater than or equal to EL9 (CentOS Stream 9 / RHEL 9 / Rocky 9 / AlmaLinux 9)

```bash
env DOCKERBUILD=1 ./pullsrc.sh
docker build -t $BUILD_TAG --platform $BUTILD_PLATFORM -f ./docker/Dockerfile.centos-stream --build-arg VERSION_NUM=$BUILD_VERSION_NUM --build-arg MIRROR=$BUILD_MIRROR .
docker run --rm -v .:/data $BUILD_TAG
```

## Build Arguments

| Argument          | Values | Description |
|-------------------|--------|-----------|
| `MIRROR`    | 0 or 1 | Set to `1` if you are in China and want to use faster domestic mirrors |
| `VERSION_NUM`     | 6,7,8,9| Specifies the target EL version (used in most Dockerfiles) |

**Example for users in China:**

Add `--build-arg MIRROR=1` to the `docker build` command.

## Output Location

After each successful build, the RPM packages are copied to:

```
./output/
```

Typical output structure:

```
output/
├── el5/
├── el6/
├── el7/
├── el8/
├── el9/
```

Each subdirectory contains the generated `.rpm` files (including debuginfo if available).

## Troubleshooting

- **Slow downloads**: Use `MIRROR=1`
- **Permission issues**: Run `chown -R $USER output/` after building
- **Docker build fails on first run**: This is normal — it needs to download base images and dependencies
- **ARM64 builds**: Requires a machine with ARM64 support or Docker Buildx multi-platform enabled

