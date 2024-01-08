# Proposal A - Runtime Annotations

This proposal recommends changes that allow runtimes to quickly select the appropriate image from an index using the platform and annotations on the descriptor.

## Links

| Description | Link |
|-------------|------|
| Image Index Spec | <https://github.com/opencontainers/image-spec/blob/main/image-index.md> |

## Modifications

### Image Spec

The following language is already included in the Image Index definition:

> If multiple manifests match a client or runtime's requirements, the first matching entry SHOULD be used.

This would add a section for "Annotations":

> Runtimes MAY be configured to prefer manifests with specific annotations.

An example would also be added to the Image Index:

```jsonc
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.index.v1+json",
  "manifests": [
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "size": 7143,
      "digest": "sha256:e692418e4cbaf90ca69d05a66403747baa33ee08806650b51fab815ad7fc331f",
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      }
    },
    {
      "mediaType": "application/vnd.oci.image.manifest.v1+json",
      "size": 7682,
      "digest": "sha256:5b0bcabd1ed22e9fb1310cf6c2dec7cdef19f0ad69efa1f392e94a4333501270",
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      },
      "annotation": {
        "com.example.gpu": "example-gpu-model"
      }
    }
  ]
}
```

In the above scenario, the generic `linux/amd64` entry is listed first and should be used by default.
Runtimes MAY be configured to prefer the `com.example.gpu: example-gpu-model` entry to run the image customized for their specific environment.

Image authors SHOULD limit annotations to those needed to differentiate descriptors or for runtimes requirements.

Runtimes MAY reject manifests that exceed size limits.
Runtimes MAY be pre-configured to search for specific annotations matching their feature set.
Runtimes are not expected to know all the annotations that they prefer for every platform they run on.
Runtimes MAY have the ability to configure a list of annotations that are preferred by that installation.
If a runtime is not configured to prefer a specific annotation, or multiple matching descriptors are encountered, the runtime SHOULD default to the first matching descriptor.

## Requirements

TBD: pending the completion of the requirements.
