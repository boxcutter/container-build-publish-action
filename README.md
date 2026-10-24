# container-build-publish-action
GitHub Action to build and publish container images with Buildx.

## Native multi-platform builds

By default the action builds every platform in the bake file's `default`
targets on one runner, emulating the others with QEMU.

To build each platform natively instead, run the action once per platform
with `platform` set, then run the `merge` action once per image to tag the
per-platform images, pushed by digest, as one multi-platform image:

```yaml
build:
  strategy:
    matrix:
      platform: [linux/amd64, linux/arm64]
      include:
        - platform: linux/amd64
          runner: ubuntu-latest
        - platform: linux/arm64
          runner: ubuntu-24.04-arm
  runs-on: ${{ matrix.runner }}
  steps:
    - uses: actions/checkout@v7
    - uses: boxcutter/container-build-publish-action@main
      with:
        workdir: hadolint
        platform: ${{ matrix.platform }}
        dockerhub-username: ${{ secrets.CONTAINER_REGISTRY_USERNAME }}
        dockerhub-password: ${{ secrets.CONTAINER_DESCRIPTION_PASSWORD }}

merge:
  needs: build
  if: ${{ github.event_name != 'pull_request' }}
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v7
    - uses: boxcutter/container-build-publish-action/merge@main
      with:
        workdir: hadolint
        dockerhub-username: ${{ secrets.CONTAINER_REGISTRY_USERNAME }}
        dockerhub-password: ${{ secrets.CONTAINER_DESCRIPTION_PASSWORD }}
```

A platform job skips the targets that don't list its platform. The merge
refuses to publish a target unless every platform it lists has a digest.
