# Proposal G - Image Compatibility Artifact

The proposal introduces a new `application/vnd.oci.image-compatibility.v1` artifact type.
Interest groups/communities SHOULD implement tools to address their specific (or common) use cases.

## Artifact Manifest

An artifact manifest MAY contain multiple blobs with different or the same media types, but MUST contain at least one.
The new artifact type for image compatibility MUST be referenced in OCI documentation.

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/vnd.oci.image-compatibility.v1",
  "config": {
    "mediaType": "application/vnd.oci.empty.v1+json",
    "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
    "size": 2
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image-compatibility.nfd.v1alpha1+json",
      "digest": "sha256:4a47f8ae4c713906618413cb9795824d09eeadf948729e213a1ba11a1e31d052",
      "size": 1710,
      "annotations": {
        "name": "profileA"
      }
    }, 
    {
      "mediaType": "application/vnd.oci.image-compatibility.nfd.v1alpha1+json",
      "digest": "sha256:35589b7dcd49cbcc48a22815aceccaf34190189a3f25c6e84772771e051a992c",
      "size": 450,
      "annotations": {
        "name": "profileB"
      }
    },
    {
      "mediaType": "application/vnd.oci.image-compatibility.nfd.v1alpha1+json",
      "digest": "sha256:494b04517cada9ba77ebdd3a94ad3e0e10a4d000dada8cb0876cd3c5e90fc826",
      "size": 2040,
      "annotations": {
        "name": "profileC"
      }
    }
  ],
  "annotations": {
    "oci.opencontainers.image.created": "2024-01-02T03:04:05Z"
  }
}
```

## Compatibility Specification

Interest groups/communities are responsible to release their own media types with corresponding schemas and overall JSON structure.
Schema MUST provide attributes that will allow to express image compatibility.
Known media types SHOULD be referenced in the OCI documentation.

## Tools implementation

Tools that implement image compatibility use cases MUST follow the schema provided by the media type.
It is recommended that additional schema extensions that solve a very specific use case be stored in annotations without interfering with the specification.
Implemented tools should be referenced in OCI documentation.

## Requirements

Based on [REQUIREMENTS.md](../REQUIREMENTS.md)

### Image Author

1. [x] As an image author, I want to update compatibility independently without having to re-release and re-distribute my image.
1. [x] As an image author, I want to have the freedom to express any compatibility that is necessary for my container to run on the host. (_Compatibility specification is defined by media types, if required groups can come with their own._)
   - [x] Including conditional compatibility, such as a container running on AMD CPU and intel CPU having different requirements.
   - [x] Including must have/nice to have compatibility.
   - [x] Matching finer grain platform definitions.
1. [ ] As an image author, I want to use a provided tool to verify the compatibility spec I wrote against the schema.
1. [ ] As an image author, I would like to create a single compatibility description that is common to a group of images.
1. [ ] As an image author, I want to be able to specify in my manifest that my multi-platform image has a compatibility spec which should be consulted.
1. [ ] As an image author, I want to be able to include compatibility specifications from base layers that I inherited from.
1. [x] As an image author, I want to ensure that runtimes without image compatibility gracefully fall back to running a usable image. (_This proposal does not affect runtimes_)

### Domain Architect

1. [x] As a domain architect I want a process to share my knowledge about \<niche topic\> compatibility with some compatibility interest group. (_Can be done over specific media type._)
1. [ ] As a domain architect I don't want to have to understand containers or develop tools for them to share this knowledge. (_Tools must be implemented by communities._)

### Tool Writer

1. [x] As a tool writer, I want to get the compatibility spec without pulling the image layer blobs.
1. [ ] As a tool writer, I would like to have a library for reading image compatibility so that I can write my own software that takes action based on the spec, e.g.: (_The library will not be created under OCI._)
   - custom k8s scheduler, admission webhooks, runtime classes etc.
1. [x] As a tool writer, compatibility validation could be integrated into non-runtime tools.
1. [x] As a tool writer, I want to be able to write tools (that use compatibility specs) that have non-standard applications (e.g., checking individual layers). (_Communities write their tools to cover their specific use cases._)

### System Runtime Administrator

1. [ ] As a system runtime administrator, I want to check whether a container is compatible with the nodes I am going to run it on using the provided tool. (_This statement is valid for all the following points: OCI will not provide tools, it is up to the community to cover the use cases._)
1. [ ] As a system runtime administrator, I would like to fetch additional documentation for understanding specific settings in the compatibility spec.
1. [ ] As a system runtime administrator, selecting which image to run should only require pulling the Index manifest, and parsing the descriptors listed.
  Additional API calls to the registry should not be required. (_Image selection by runtimes is out of scope for this proposal_)
1. [ ] As a system runtime administrator, I want to validate whether all running applications in the cluster are compatible with a new operating system (or new operating system version) or not before migrating.
1. [ ] As a system runtime administrator, I want to validate whether all running applications in the cluster are compatible with new hardware (cpu, gpu, nic etc.) or not before migrating. (_The same note as above_)
1. [ ] As a system runtime administrator, I want to use annotations to schedule for the appropriate resources.
1. [ ] As a system runtime administrator, the runtime should not need to know about all possible types of hardware.
   Perhaps hooks could be added for users to inject their own image selection criteria on a given host, or annotations could be injected.

### Deployment Engineer

1. [ ] As a deployment engineer, I want to parse an image index and find the “optimal” image for the cluster node I am aiming to run the image on.
That includes being able to: (_This statement is valid for all the following points: OCI will not provide tools, it is up to the community to cover the use cases._)
   - Discover the image that fits the selected host.
   - Find the best match from the nodes and images I have available.
   - Determine that the image is not fitting the selected host.
1. [ ] As a deployment engineer, I want the image compatibility check to be performed without downloading or executing the referenced image layers.
1. [ ] As a deployment engineer, I should be able to add the compatibility to images already being used in production, especially for the images released before image compatibility wg was created.
1. [ ] As a deployment engineer, I want to be able to specify the version and variant of an application or other user specific configuration (e.g. MPI), and not only hardware/kernel details in the compatibility specification.
1. [ ] As a deployment engineer, I want my compatibility spec runtime to be able to select the best possible runtime available on a node (e.g. runc vs nvidia vs wasm).
1. [ ] As a deployment engineer, I want to be able to rank the images in a multi-platform image so that the runtime can know which one to choose when more than one image is compatible with the runtime environment.
1. [ ] As a deployment engineer, I want to reuse community projects so that I don't duplicate and integrate the functionality of the already existing tools.

### Registry Maintainer

1. [ ] As a registry user or operator I want to have a common way of inspecting information about image compatibility to enable users to find an image that best matches their system.
1. [x] As a registry operator (or user that has no control over their registry implementation), I want any compatibility changes to not depend on registry server changes or upgrades.

### OCI Specification Maintainer

1. [x] As a spec maintainer, I want the solution to avoid breaking other specs (confidential images, image signing, existing implementations for runtimes picking images).
1. [x] As a spec maintainer, I don't want the spec to update for new hardware devices, kernel releases, or other external dependencies.
1. [x] As a spec maintainer, I don't want to overlap significantly with solutions from other specs (like SBOMs).

### Security Administrator

1. [x] As a security administrator, I want predictable behavior from runtimes, which does not change based on unsigned content.
1. [ ] As a security administrator, I want to ensure the compatibility spec cannot be used to escalate privileges beyond what is requested by the deployment engineer. (_This really depends on the implementation provided by community._)
1. [x] As a security (or more generally, XYZ) researcher, I want to annotate a container (separately) with my niche jargon of metadata.
1. [x] As a security administrator, I want to know that image compatibility cannot be used to circumvent image signing.
1. [ ] As a security administrator I want to catalog SBOMs of compatible images to put into a report about software used by my group / institution.
