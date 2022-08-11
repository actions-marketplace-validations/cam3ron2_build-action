# Build and Push

A collection of actions to build and push a multiarch Docker image aggregated into one action.

This action will build an image with Docker buildx for amd64 and arm64, and then push it to the registry on a merged pull request. Tags are automatically generated for the image based on metadata in the repo such as the commit sha. It also makes use of actions cache to minimize build time, and ouputs imageid, digest, and metadata for consumption by later steps.

- [Build and Push](#build-and-push)
  - [Usage](#usage)
  - [Inputs](#inputs)
  - [Outputs](#outputs)

## Usage

```yaml
name: "Build and Push"

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  build_test:
    name: Build
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      - name: Build and Push
        uses: cam3ron2/build-action@v1
        with:
          username: ${{ github.repository_owner }}
          password: ${{ secrets.GITHUB_TOKEN }}
          registry: ghcr.io
          repository: cam3ron2/example-app
```

## Inputs

Required Inputs:

```yaml
repository:
  description: 'Repository to push to'
  default: None
  required: true
username:
  description: 'Username for the registry'
  default: None
  required: true
password:
  description: 'Password for the registry'
  default: None
  required: true
```

Optional Inputs:

```yaml
file:
  description: 'Path to the Dockerfile'
  default: 'Dockerfile'
  required: false
context:
  description: "Build's context is the set of files located in the specified PATH or URL"
  default: None
  required: false
build-args:
  description: 'Arguments to pass to the Dockerfile'
  default: None
  required: false
registry:
  description: 'Registry that houses the repository'
  default: None
  required: false
push:
  description: 'Push the image to the registry'
  default: 'true'
  required: false
publish-cache:
  description: 'If true, create an artifact'
  default: 'true'
  required: false
```

## Outputs

- imageid: The image id of the built image. ex: `sha256:someshahere`
- digest: The digest of the built image. ex: `sha256:someshahere`
- metadata: A json object containing the metadata of the built image, including the tags that were pushed. Example:

```json
  {
    "cache.manifest": "{\"mediaType\":\"application/vnd.oci.image.index.v1+json\",\"digest\":\"sha256:someshahere\",\"size\":1234}",
    "containerimage.buildinfo/linux/amd64": {
      "frontend": "dockerfile.v0",
      "attrs": {
        "filename": "Dockerfile"
      },
      "sources": [
        {
          "type": "docker-image",
          "ref": "docker.io/library/alpine:latest",
          "pin": "sha256:someshahere"
        },
        {
          "type": "docker-image",
          "ref": "docker.io/library/golang:latest",
          "pin": "sha256:someshahere"
        }
      ]
    },
    "containerimage.buildinfo/linux/arm64": {
      "frontend": "dockerfile.v0",
      "attrs": {
        "filename": "Dockerfile"
      },
      "sources": [
        {
          "type": "docker-image",
          "ref": "docker.io/library/alpine:latest",
          "pin": "sha256:someshahere"
        },
        {
          "type": "docker-image",
          "ref": "docker.io/library/golang:latest",
          "pin": "sha256:someshahere"
        }
      ]
    },
    "containerimage.descriptor": {
      "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
      "digest": "sha256:someshahere",
      "size": 743
    },
    "containerimage.digest": "sha256:someshahere",
    "image.name": "ghcr.io/cam3ron2/example-app:pr-2,ghcr.io/cam3ron2/example-app:sha-7bab380"
  }
```
