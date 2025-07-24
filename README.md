# ERPNext Custom Docker Image Build

A custom ERPNext Docker image build configuration that includes additional Frappe applications for extended functionality.

## Overview

This project builds a custom ERPNext Docker image with the following applications:

- **ERPNext** - Open source ERP system
- **HRMS** - Human Resource Management System
- **Raven** - Team communication and collaboration tool
- **Builder** - Website and page builder
- **Frappe** - Core framework for building web applications

## Features

- **Automated Builds**: GitHub Actions workflow that builds and deploys images automatically
- **Version Management**: Automatic semantic versioning for Docker images
- **Scheduled Updates**: Automated builds every 6 hours to ensure the latest versions
- **Multi-App Integration**: Seamlessly integrates multiple Frappe applications

## Docker Image

The built images are available on Docker Hub:
- **Repository**: `mitexleo/syserp`
- **Tags**: `latest` (most recent build) and semantic versions (e.g., `1.0.0`, `1.0.1`, `1.0.2`)

## Build Configuration

### Apps Configuration

The `apps.json` file defines which Frappe applications are included in the build. This file is used to specify the repositories and branches for each application:

```json
[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/hrms",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/frappe",
    "branch": "version-15"
  }
  // ... other apps
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
- **Scheduled**: Every 6 hours
- **Manual**: Via GitHub Actions workflow dispatch

### Versioning

The system automatically:
1. Checks for new releases every 6 hours
2. Increments the patch version (e.g., 1.0.0 → 1.0.1)
3. Tags the new image with both the new version and `latest`

## Development

### Prerequisites

- Docker and Docker Compose
- GitHub account with repository access
- Docker Hub account for image registry
- `jq` and `curl` for local testing

### Local Development

1. Clone the repository:
   ```bash
   git clone <your-repo-url>
   cd ERPNext
   ```

2. Modify `apps.json` as needed

3. Test the build locally:
   ```bash
   # Clone frappe_docker
   git clone https://github.com/frappe/frappe_docker

   # Build the image
   cd frappe_docker
   APPS_JSON_BASE64=$(base64 -w 0 ../apps.json)
   docker build \
     --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe \
     --build-arg=FRAPPE_BRANCH=version-15 \
     --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 \
     --tag=local-erpnext:test \
     --file=images/layered/Containerfile .

   # Run the container
   docker run -d -p 8000:8000 local-erpnext:test
   ```

### Environment Variables

Configure the following secrets in your GitHub repository:

- `DOCKER_USERNAME`: Your Docker Hub username
- `DOCKER_PASSWORD`: Your Docker Hub password or access token
- `GITHUB_TOKEN`: Your GitHub token for repository access

## Troubleshooting

### Common Issues

1. **Build Failures**: Check that all apps in `apps.json` are accessible and branches exist. Ensure `jq` and `curl` are installed.
2. **Version Conflicts**: Ensure all apps are compatible with the specified Frappe version.
3. **Docker Hub Permissions**: Verify Docker Hub credentials are correctly set in GitHub secrets.
4. **GitHub Token**: Ensure the GitHub token has the necessary permissions for repository access.

### Logs

View container logs:
```bash
docker logs <container-name>
```

View build logs in GitHub Actions for detailed build information.

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
- **Build workflow**: Check the workflow file `.github/workflows/build.yml` for configuration details

## Related Links

- [ERPNext Documentation](https://docs.erpnext.com/)
- [Frappe Framework](https://frappeframework.com/)
- [Frappe Docker](https://github.com/frappe/frappe_docker)
- [Docker Hub Repository](https://hub.docker.com/r/mitexleo/syserp)
- [GitHub Actions Workflow](https://github.com/your-username/ERPNext/blob/main/.github/workflows/build.yml)
