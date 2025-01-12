# KMP Publish Android App to Play Store Action

This GitHub Action automates the process of publishing Android applications to the Google Play Store Beta or Internal testing track. It handles building App Bundles (AAB), signing, and deployment through Fastlane.

## Features

- Automated Play Store deployment
- Support for Internal and Beta testing tracks
- AAB (Android App Bundle) generation
- Release signing
- Artifact archiving
- Google Play Console integration

## Prerequisites

Before using this action, ensure you have:

1. A Google Play Console account with an existing app
2. A service account with appropriate permissions in Google Play Console
3. A release keystore for signing your app
4. Google Services JSON file for your project

## Setup

### Fastlane Setup

Create a `Gemfile` in your project root:

```ruby
source "https://rubygems.org"

gem "fastlane"
```

Create a `fastlane/Fastfile` with the following content:

```ruby
default_platform(:android)

platform :android do
  desc "Bundle Play Store Release"
  lane :bundlePlayStoreRelease do |options|
    gradle(
      task: "bundle",
      build_type: "Release",
      properties: {
        "android.injected.signing.store.file" => options[:storeFile],
        "android.injected.signing.store.password" => options[:storePassword],
        "android.injected.signing.key.alias" => options[:keyAlias],
        "android.injected.signing.key.password" => options[:keyPassword],
      }
    )
  end

  desc "Deploy to Internal Testing Track"
  lane :deployInternal do
    upload_to_play_store(
      track: 'internal',
      aab: lane_context[SharedValues::GRADLE_AAB_OUTPUT_PATH],
      json_key: 'secrets/playStorePublishServiceCredentialsFile.json',
      skip_upload_metadata: true,
      skip_upload_images: true,
      skip_upload_screenshots: true,
      skip_upload_changelogs: true
    )
  end

  desc "Promote Internal to Beta"
  lane :promote_to_beta do
    upload_to_play_store(
      track: 'internal',
      track_promote_to: 'beta',
      json_key: 'secrets/playStorePublishServiceCredentialsFile.json',
      skip_upload_metadata: true,
      skip_upload_images: true,
      skip_upload_screenshots: true,
      skip_upload_changelogs: true
    )
  end
end
```

## Usage

Add the following workflow to your GitHub Actions:

```yaml
name: Publish to Play Store

on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Publish to Play Store
        uses: openMF/kmp-publish-android-on-playstore-beta-action@v2.0.0
        with:
          android_package_name: 'app'
          release_type: 'beta'  # or 'internal'
          google_services: ${{ secrets.GOOGLE_SERVICES }}
          playstore_creds: ${{ secrets.PLAYSTORE_CREDS }}
          keystore_file: ${{ secrets.RELEASE_KEYSTORE }}
          keystore_password: ${{ secrets.KEYSTORE_PASSWORD }}
          keystore_alias: ${{ secrets.KEYSTORE_ALIAS }}
          keystore_alias_password: ${{ secrets.KEYSTORE_ALIAS_PASSWORD }}
```

## Inputs

| Input                     | Description                                         | Required |
|---------------------------|-----------------------------------------------------|----------|
| `release_type`            | Type of release ('internal' or 'beta')              | Yes      |
| `android_package_name`    | Name of your Android project module                 | Yes      |
| `keystore_file`           | Base64 encoded release keystore file                | Yes      |
| `keystore_password`       | Password for the keystore file                      | Yes      |
| `keystore_alias`          | Alias for the keystore file                         | Yes      |
| `keystore_alias_password` | Password for the keystore alias                     | Yes      |
| `google_services`         | Base64 encoded Google services JSON file            | Yes      |
| `playstore_creds`         | Base64 encoded Play Store service account JSON file | Yes      |

## Setting up Secrets

1. Encode your files to base64:
```bash
base64 -i path/to/release.keystore -o keystore.txt
base64 -i path/to/google-services.json -o google-services.txt
base64 -i path/to/play-store-credentials.json -o playstore-creds.txt
```

2. Add the following secrets to your GitHub repository:
- `RELEASE_KEYSTORE`: Content of keystore.txt
- `KEYSTORE_PASSWORD`: Your keystore password
- `KEYSTORE_ALIAS`: Your keystore alias
- `KEYSTORE_ALIAS_PASSWORD`: Your keystore alias password
- `GOOGLE_SERVICES`: Content of google-services.txt
- `PLAYSTORE_CREDS`: Content of playstore-creds.txt

## Release Process

1. The action will:
   - Build a signed Android App Bundle (AAB)
   - Upload the AAB as a GitHub artifact
   - Deploy to Play Store Internal testing track
   - Optionally promote to Beta if `release_type` is set to 'beta'

## Artifacts

The action saves the generated AAB files as artifacts with the name 'playstore-aabs'. You can find these in your GitHub Actions run.
