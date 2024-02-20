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

Image authors SHOULD limit annotations in the manifest list descriptors to those needed to differentiate descriptors or for other runtime requirements.

Runtimes MAY reject manifests that exceed size limits.
Runtimes MAY be pre-configured to search for specific annotations matching their feature set.
Runtimes are not expected to know all the annotations that they prefer for every platform they run on.
Runtimes MAY have the ability to configure a list of annotations that are preferred by that installation.
Runtime preferences MAY be weighted and computed as the sum of each of the annotation weights.
If a runtime is not configured to prefer a specific annotation, or multiple matching descriptors are encountered, the runtime SHOULD default to the first matching descriptor.

## Requirements

### Image Author

- [x] As an image author, I want to update compatibility independently without having to re-release and re-distribute my image. _(This would recreate the index, changing its digest, but not the individual listed images.)_
- [x] As an image author, I want to have the freedom to express any compatibility that is necessary for my container to run on the host.
  - Including conditional compatibility, such as a container running on AMD CPU and intel CPU having different requirements.
  - Including must have/nice to have compatibility.
  - Matching finer grain platform definitions.
- [ ] As an image author, I want to use a provided tool to verify the compatibility spec I wrote against the schema. _(Tooling would be provided by the existing build and runtime tooling authors for their own projects.)_
- [x] As an image author, I would like to create a single compatibility description that is common to a group of images.
- [x] As an image author, I want to be able to specify in my manifest that my multi-platform image has a compatibility spec which should be consulted.
- [x] As an image author, I want to be able to include compatibility specifications from base layers that I inherited from. _(Dependent on build tooling functionality.)_
- [x] As an image author, I want to ensure that runtimes without image compatibility gracefully fall back to running a usable image.

### Domain Architect

- [x] As a domain architect I want a process to share my knowledge about \<niche topic\> compatibility with some compatibility interest group.
- [x] As a domain architect I don't want to have to understand containers or develop tools for them to share this knowledge. _(Adding compatibility annotations would be a feature of the container image build tooling.)_

### Tool Writer

- [x] As a tool writer, I want to get the compatibility spec without pulling the image layer blobs.
- [x] As a tool writer, I would like to have a library for reading image compatibility so that I can write my own software that takes action based on the spec, e.g.: _(Tooling already exists to read the OCI Index and annotations on the descriptor.)_
  - custom k8s scheduler, admission webhooks, runtime classes etc.
- [x] As a tool writer, compatibility validation could be integrated into non-runtime tools.
- [x] As a tool writer, I want to be able to write tools (that use compatibility specs) that have non-standard applications (e.g., checking individual layers).

### System Runtime Administrator

- [ ] As a system runtime administrator, I want to check whether a container is compatible with the nodes I am going to run it on using the provided tool. _(Tooling would be created and maintained by others implementing the OCI specs.)_
- [ ] As a system runtime administrator, I would like to fetch additional documentation for understanding specific settings in the compatibility spec. _(Annotation documentation would be managed by projects responsible for their namespace following the reverse DNS syntax.)_
- [x] As a system runtime administrator, selecting which image to run should only require pulling the Index manifest, and parsing the descriptors listed.
  Additional API calls to the registry should not be required.
- [ ] As a system runtime administrator, I want to validate whether all running applications in the cluster are compatible with a new operating system (or new operating system version) or not before migrating. _(This would require third party tooling, or building a second cluster to test before blue/green migrating.)_
- [ ] As a system runtime administrator, I want to validate whether all running applications in the cluster are compatible with new hardware (cpu, gpu, nic etc.) or not before migrating. _(This would require third party tooling, or building a second cluster to test before blue/green migrating.)_
- [x] As a system runtime administrator, I want to use annotations to schedule for the appropriate resources.
- [x] As a system runtime administrator, the runtime should not need to know about all possible types of hardware.
  Perhaps hooks could be added for users to inject their own image selection criteria on a given host, or annotations could be injected.

### Deployment Engineer

- [x] As a deployment engineer, I want to parse an image index and find the “optimal” image for the cluster node I am aiming to run the image on.
That includes being able to:
  - Discover the image that fits the selected host.
  - Find the best match from the nodes and images I have available.
  - Determine that the image is not fitting the selected host.
- [x] As a deployment engineer, I want the image compatibility check to be performed without downloading or executing the referenced image layers.
- [x] As a deployment engineer, I should be able to add the compatibility to images already being used in production, especially for the images released before image compatibility wg was created. _(This would recreate the index, changing its digest, but not the individual listed images.)_
- [x] As a deployment engineer, I want to be able to specify the version and variant of an application or other user specific configuration (e.g. MPI), and not only hardware/kernel details in the compatibility specification.
- [x] As a deployment engineer, I want my compatibility spec runtime to be able to select the best possible runtime available on a node (e.g. runc vs nvidia vs wasm). _(Theoretically possible, but would require implementation effort by projects like containerd.)_
- [x] As a deployment engineer, I want to be able to rank the images in a multi-platform image so that the runtime can know which one to choose when more than one image is compatible with the runtime environment.
- [x] As a deployment engineer, I want to reuse community projects so that I don't duplicate and integrate the functionality of the already existing tools.

### Registry Maintainer

- [x] As a registry user or operator I want to have a common way of inspecting information about image compatibility to enable users to find an image that best matches their system.
- [x] As a registry operator (or user that has no control over their registry implementation), I want any compatibility changes to not depend on registry server changes or upgrades.

### OCI Specification Maintainer

- [x] As a spec maintainer, I want the solution to avoid breaking other specs (confidential images, image signing, existing implementations for runtimes picking images). _(Signing would be broken if the compatibility definitions are changed by someone other than the image author, requiring the image to be resigned by a trusted identity.)_
- [x] As a spec maintainer, I don't want the spec to update for new hardware devices, kernel releases, or other external dependencies.
- [x] As a spec maintainer, I don't want to overlap significantly with solutions from other specs (like SBOMs).

### Security Administrator

- [x] As a security administrator, I want predictable behavior from runtimes, which does not change based on unsigned content.
- [x] As a security administrator, I want to ensure the compatibility spec cannot be used to escalate privileges beyond what is requested by the deployment engineer.
- [x] As a security (or more generally, XYZ) researcher, I want to annotate a container (separately) with my niche jargon of metadata. _(Outside of the scope of the compatibility WG, this is already possible in the image-spec.)_
- [x] As a security administrator, I want to know that image compatibility cannot be used to circumvent image signing. _(Signing would be broken if the compatibility definitions are changed by someone other than the image author, requiring the image to be resigned by a trusted identity.)_
- [x] As a security administrator I want to catalog SBOMs of compatible images to put into a report about software used by my group / institution. _(SBOMs are outside of the scope of the compatibility WG. Generation, distribution, and consumption of SBOMs would continue to work as already provided by those specs.)_
