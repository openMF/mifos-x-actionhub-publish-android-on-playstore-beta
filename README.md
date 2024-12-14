# KMP Publish Android App to Play Store Beta

This GitHub Action automates the process of building and publishing Android applications to the Google Play Store's Internal Testing track, with an option to promote to Beta. It handles the complete workflow from setting up the build environment to managing the Play Store deployment using Fastlane.

## Features

- Configurable release type (internal/beta)
- Automated version code generation
- Support for signing configuration
- Play Store credential management
- Gradle caching for faster builds
- Automated internal to beta promotion

## Prerequisites

- A valid Android project with Gradle configuration
- Google Play Store developer account
- Keystore file for app signing
- Google Services configuration
- Play Store credentials

## Inputs

| Input                  | Description                            | Required |
|------------------------|----------------------------------------|----------|
| `release_type`         | Type of release (internal/beta)        | Yes      |
| `android_package_name` | Name of the Android project module     | Yes      |
| `keystore_file`        | Base64 encoded keystore file           | Yes      |
| `keystore_password`    | Password for the keystore file         | Yes      |
| `key_alias`            | Key alias for the keystore file        | Yes      |
| `key_password`         | Password for the key alias             | Yes      |
| `google_services`      | Google services JSON file content      | Yes      |
| `playstore_creds`      | Firebase credentials JSON file content | Yes      |

## Usage

```yaml
jobs:
  publish:
    runs-on: macos-latest
    steps:
      - uses: openMF/kmp-publish-android-on-playstore-beta-action@v1.0.0
        with:
          release_type: 'beta'
          android_package_name: 'app'
          keystore_file: ${{ secrets.KEYSTORE_FILE }}
          keystore_password: ${{ secrets.KEYSTORE_PASSWORD }}
          key_alias: ${{ secrets.KEY_ALIAS }}
          key_password: ${{ secrets.KEY_PASSWORD }}
          google_services: ${{ secrets.GOOGLE_SERVICES }}
          playstore_creds: ${{ secrets.PLAYSTORE_CREDS }}
```

## Workflow Details

1. **Environment Setup**
    - Configures Java 17 (Zulu distribution)
    - Sets up Gradle with caching
    - Installs Ruby and Fastlane with required plugins

2. **Version Management**
    - Automatically generates version codes based on commit history
    - Reads version name from version.txt

3. **Secret Management**
    - Decodes and configures the keystore file
    - Sets up Google Services configuration
    - Configures Play Store credentials

4. **Build Process**
    - Builds a signed Android App Bundle (AAB)
    - Archives the AAB as a build artifact

5. **Deployment**
    - Deploys to Play Store Internal Testing track
    - Optionally promotes to Beta if specified

## Requirements

- GitHub Actions runner with Ubuntu
- Gradle-based Android project
- Fastlane configuration in the repository
- Required secrets configured in repository settings

## Notes

- The action uses Gradle's version file generation
- Version code is calculated using: `(commits + tags) << 1`
- Artifacts are automatically uploaded and available in the GitHub Actions interface
- The action supports both internal testing and beta track deployments
- Gradle caching is enabled to improve build performance

## Error Handling

The action will fail if:
- Any required input is missing
- The keystore file is invalid
- The build process fails
- Play Store deployment encounters errors
- Promotion to beta fails (if specified)

## Security Considerations

- All sensitive inputs should be stored as GitHub Secrets
- The keystore file must be base64 encoded
- Credentials are handled securely and not exposed in logs