# ERPNext Custom Docker Image Build

A custom ERPNext Docker image build configuration that includes multiple Frappe applications for extended functionality with optimized build performance.

## Overview

This project builds a custom ERPNext Docker image with the following applications:

- **Frappe** (core framework) - version-16
- **ERPNext** - version-16
- **HRMS** - Human Resource Management System - version-16
- **Builder** - Website and page builder - master branch
- **CRM** - Customer Relationship Management - main branch
- **Helpdesk** - Ticketing and support system - main branch
- **Telephony** - Telephony integration - develop branch
- **Mail** - Email management - develop branch

The image is built using GitHub Actions with performance optimizations and published to Docker Hub as `mitexleo/syserp`.

## Features

- **Automated Builds**: GitHub Actions workflow that builds and deploys images automatically
- **Build Optimizations**: Docker BuildKit, layer caching, and automatic cleanup for smaller images and faster builds
- **Version Management**: Automatic semantic versioning for Docker images
- **Scheduled Updates**: Automated builds every 6 hours to check for new releases
- **Multi-App Integration**: Seamlessly integrates multiple Frappe applications
- **Release Monitoring**: Automatically detects new releases from all included repositories

## Build Optimizations

The build process includes several optimizations for reduced image size and faster builds:

### Image Size Reduction
- **Multi-stage Build**: Uses the upstream Frappe layered image build, which keeps the final image lean
- **Automatic Cleanup**: The upstream Containerfile already removes `.git` directories, pip caches, and build dependencies
- **Expected Reduction**: 200-500MB smaller images compared to a single-stage build

### Build Speed Improvements
- **GitHub Actions Cache**: Uses `type=gha` for efficient layer reuse between builds, stored in GitHub's managed cache
- **BuildKit Secrets**: Apps are installed via Docker secrets (not build args), preventing token leaks and enabling proper layer caching
- **CACHE_BUST Control**: Cache invalidation is controlled via the `CACHE_BUST` build arg, which changes only when tracked repos or `apps.json` are updated
- **Expected Speedup**: 50-70% faster builds using GHA cache

## Docker Image

The built images are available on Docker Hub:
- **Repository**: `mitexleo/syserp`
- **Tags**: `latest` (most recent build) and semantic versions (e.g., `1.0.0`, `1.0.1`)
- **Base**: Built on Frappe version-16 framework
- **Optimized**: Includes build optimizations for smaller size and faster deployment

## Build Configuration

### Apps Configuration

The `apps.json` file defines which Frappe applications are included in the build:

```json
[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-16"
  },
  {
    "url": "https://github.com/frappe/hrms",
    "branch": "version-16"
  },
  {
    "url": "https://github.com/frappe/builder",
    "branch": "master"
  },
  {
    "url": "https://github.com/frappe/crm",
    "branch": "main"
  },
  {
    "url": "https://github.com/frappe/helpdesk",
    "branch": "main"
  },
  {
    "url": "https://github.com/frappe/telephony",
    "branch": "develop"
  },
  {
    "url": "https://github.com/frappe/mail",
    "branch": "develop"
  }
]
```

### Adding New Apps

To add a new Frappe application, follow these steps:

1. Edit `apps.json` and add your app configuration:
   ```json
   {
     "url": "https://github.com/your-username/your-app",
     "branch": "main"
   }
   ```

2. Commit and push your changes

The GitHub Actions workflow will automatically build a new image with your app included. The workflow checks for new releases every 6 hours and updates the image accordingly.

## Automated Builds

### Triggers

The build workflow runs automatically:
- **Scheduled**: Every 6 hours (checks for new releases)
- **Manual**: Via GitHub Actions workflow dispatch (force build option)

### Versioning

The system automatically:
1. Checks for new releases from all monitored repositories
2. Increments the patch version (e.g., 1.0.0 → 1.0.1) when changes are detected
3. Tags the new image with both the new version and `latest`
4. Updates `REPO_VERSIONS.json` with the latest release tags

### Optimization Process

During each build:
1. **BuildKit Setup**: Configures Docker Buildx with QEMU for cross-platform support
2. **GitHub Actions Cache**: Pulls cached layers from GHA cache for faster rebuilds
3. **Secret-based Build**: Apps are passed via BuildKit secrets (not build args), keeping tokens secure
4. **Optimized Build**: Uses BuildKit with GHA cache (`type=gha`) for efficient layer reuse

## Development

### Prerequisites

- Docker and Docker Compose
- GitHub account with repository access
- Docker Hub account for image registry
- `jq` and `curl` for local testing

### Local Development

1. Clone the repository:
   ```bash
   git clone https://github.com/mitexleo/ERPNext
   cd ERPNext
   ```

2. Modify `apps.json` as needed

3. Test the build locally:
   ```bash
   # Clone frappe_docker
   git clone https://github.com/frappe/frappe_docker

   # Build the image (Docker Engine v23+ required for BuildKit secrets)
   docker build \
     --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
     --build-arg=FRAPPE_BRANCH=version-16 \
     --secret=id=apps_json,src=apps.json \
     --tag=local-erpnext:test \
     --file=frappe_docker/images/layered/Containerfile .

   # Run the container
   docker run -d -p 8000:8000 local-erpnext:test
   ```

   > **Note:** The `apps.json` is passed as a [BuildKit secret](https://docs.docker.com/build/building/secrets/), not a build-arg. This ensures private repository tokens are never visible in `docker image history`. Requires Docker Engine v23+.

### Environment Variables

Configure the following secrets in your GitHub repository:

- `DOCKER_USERNAME`: Your Docker Hub username
- `DOCKER_PASSWORD`: Your Docker Hub password or access token
- `GITHUB_TOKEN`: Your GitHub token for repository access

## Troubleshooting

### Common Issues

1. **Build Failures**: Check that all apps in `apps.json` are accessible and branches exist. Ensure `jq` and `curl` are installed.
2. **Version Conflicts**: Ensure all apps are compatible with Frappe version-16.
3. **Docker Hub Permissions**: Verify Docker Hub credentials are correctly set in GitHub secrets.
4. **GitHub Token**: Ensure the GitHub token has the necessary permissions for repository access.
5. **Release Detection**: Check `REPO_VERSIONS.json` to see which releases are being tracked.
- **Cache Issues**: If builds are slow, the GitHub Actions cache may need to be cleared. Go to Actions → Manage caches in your repository settings.

### Optimization Notes

- **First Build**: May be slower as it builds the cache
- **Subsequent Builds**: Should be significantly faster with cached layers
- **Cache Persistence**: GHA cache persists based on LRU policy and has a repository size limit
- **Manual Cache Clear**: Go to repository Settings → Actions → Caches to delete cached layers
- **Cache Busting**: Use `force_build` workflow dispatch trigger to force a full rebuild

### Logs

View container logs:
```bash
docker logs <container-name>
```

View build logs in GitHub Actions for detailed build information and optimization status.

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make your changes to `apps.json` or workflow files
4. Commit your changes: `git commit -am 'Add new feature'`
5. Push to the branch: `git push origin feature/your-feature`
6. Create a Pull Request

## License

This project configuration is open source. Individual applications included may have their own licenses.

## Support

For issues related to:
- **This build configuration**: Create an issue in this repository
- **ERPNext functionality**: Visit [ERPNext Community](https://discuss.frappe.io/)
- **Individual apps**: Refer to their respective repositories
- **Build optimizations**: Check `.github/workflows/build.yml` for configuration details
- **Docker issues**: Review Docker Hub repository logs

## Related Links

- [ERPNext Documentation](https://docs.erpnext.com/)
- [Frappe Framework](https://frappeframework.com/)
- [Frappe Docker](https://github.com/frappe/frappe_docker)
- [Docker Hub Repository](https://hub.docker.com/r/mitexleo/syserp)
- [GitHub Actions Workflow](https://github.com/mitexleo/ERPNext/blob/main/.github/workflows/build.yml)