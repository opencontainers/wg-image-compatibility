# Proposal I - compatibilities extension in image index platform and image manifest config

This proposal is born from proposal A to H.

The purpose of the proposal, based on the maintainers feedback, is to address the concern maintaining a standalone artifact specification in OCI.

## image spec platform object extension

### revive platform.features

platform.features will be reused for image selection, refer to [proposal E](https://github.com/opencontainers/wg-image-compatibility/blob/main/docs/proposals/PROPOSAL_E.md)

### add platform.compatibilities descriptor

Add a new descriptor in platform object in [image index](https://github.com/opencontainers/image-spec/blob/main/image-index.md#image-index-property-descriptions).

- **`manifests`** _array of objects_

  - **`platform`** _object_

    This OPTIONAL property describes the minimum runtime requirements of the image.
    This property SHOULD be present if its target is platform-specific.

    - **`architecture`** _string_

    - **`variant`** _string_

    - **`os`** _string_

    - **`os.version`** _string_

    - **`os.features`** _array of strings_

    - **`features`** _array of strings_

    - **`compat`** _[descriptor](https://github.com/opencontainers/image-spec/blob/main/descriptor.md)_

      This OPTIONAL property specifies a descriptor of a compatibilities manifest.

## Compatibilities Manifest

The compatibilities manifest is a special image manifest, contains only config and no blob layer.

```json
{
  "schemaVersion": 2,
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "config": {
    "mediaType": "application/vnd.oci.image.config.v1+json",
    "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
    "size": 678
  },
  "layers": [],
  "annotations": {
    "oci.opencontainers.image.created": "2024-03-27T08:08:08Z"
  }
}
```

### Config Extension

The config object will be extened with compatibilities field.

The required rootfs COULD be empty if the config contains compatibilities field.

- **`compatibilities`** _array of compatibility sets_

  This REQUIRED property describes all image compatibility sets, contains at least one compatibility set.

  - **`compatibility set`** _array of compatibility requirements_

    This REQUIRED property describes compatibility requirements in one compatibility set.

    If one of the compatibility sets passes the compatibility check against the target node, that means the node is compatible with the image.

    There are three category key-value pair inside the compatibility set.

    - **`compatibility label`** _string key value pair_

      This REQUIRED property describes a feature compatibility requirement on the target host, the label accepts OCI and community defined labels.

      OCI will only maintain standardized compatility lable proposed by communities.

      The value of compatibility label SHOULD be described in contiguous or discontinuous range if the value discovered on nodes can be different.

      It's the tool's responsibility to interpret the value of OCI or community defined label, and matching criteria with the value discovered from the node via NFD tool or other tool. NFD means the Kubernetes node feature discovering SIG.

      Inevitably, some compatibility fields MAY be duplicated among compatibility sets.

      The value type of this field is STRING.

    - **`tags`** _array of string_

      This OPTIONAL property allows to group compatibility sets by tags.

      The value type of this field is STRING.

    - **`description`**

      This OPTIONAL property is to describe human-readable information on a compatibility set.

      The value type of this field is STRING.

### Config Example

All labels in this section are only for examples.

For example image is compatible with different hardware combinations

```json
{
    "created": "2024-06-11T22:22:56.015925234Z",
    "author": "ChaoyiHuang<joehuang.sweden@gmail.com>",
    "config": {
        "compatibilities": [
            {
                "oci.cpu.vendor": "GenuineIntel",
                 "oci.cpu.features": "AVX512FP16",
                 "oci.kernel.configurations": "PREEMPT",
                 "oci.os.glibc": ">=2.31, <=2.37",
                 "oci.pci.devices": "15B3.020D",
                 "tags": "intel"
            },
            {
                 "oci.cpu.vendor": "AuthenticAMD",
                 "oci.cpu.features": "FPHP",
                 "oci.kernel.configurations": "PREEMPT",
                 "oci.os.glibc": ">=2.31, <=2.37",
                 "oci.pci.devices": "1002.67ff",
                 "tags": "amd",
                 "description": "works only with AMD CPU and AMD GPU"
            }]
    },
    "rootfs": {}
}
```

## Major concerns to address

The path to retrieve compatibilities is: image index -> image-manifest entry -> compat descriptor -> image manifest for compatibilities -> config descriptor -> config -> compatibilities.

No new standard to maintain, just extension on existing specification.

Reduce the risk of image index size exceeding 4MB size limitation.

Update the compat descriptor in image index manifest entry to the new compatibilities manifest without rebuild image.

Image compatibilites validation can be done without runtime engine involved.

Runtime engine can be configured whether to pull compatibilites manifest, and whether to validate the compatibilites in container lifecycle flow.

OCI only maintains common interest compatibility labels.

## Compatibility and NFD filed labels

The proposal itself does not intend to define concrete compatibility field labels, the compatibility field labels can be proposed and accepted in OCI if necessary.

Because the value of compatibility field SHOULD be described in contiguous or discontinuous range if the value discovered on nodes can be different, NFD labels are dynamically created according to node features, it's not one to one mapping.

For example, oci.kernel.configurations may contain several boot configurations in a single string, but in NFD, several labels like kernel-config.PREEMPT, kernel-config.intel_iommu may be applied.

## Requirements

Based on [REQUIREMENTS.md](../REQUIREMENTS.md)

### Image Author

1. [x] As an image author, I want to update compatibility independently without having to re-release and re-distribute my image.
1. [x] As an image author, I want to have the freedom to express any compatibility that is necessary for my container to run on the host.
   - [x] Including conditional compatibility, such as a container running on AMD CPU and intel CPU having different requirements.
   - [x] Including must have/nice to have compatibility.
   - [x] Matching finer grain platform definitions.
1. [x] As an image author, I want to use a provided tool to verify the compatibility spec I wrote against the schema.
1. [ ] As an image author, I would like to create a single compatibility description that is common to a group of images.
1. [x] As an image author, I want to be able to specify in my manifest that my multi-platform image has a compatibility spec which should be consulted.
1. [ ] As an image author, I want to be able to include compatibility specifications from base layers that I inherited from.
1. [x] As an image author, I want to ensure that runtimes without image compatibility gracefully fall back to running a usable image. (_This proposal does not affect runtimes_)

### Domain Architect

1. [x] As a domain architect I want a process to share my knowledge about \<niche topic\> compatibility with some compatibility interest group. (_Can be done over annotations field in the proposed specification_)
1. [x] As a domain architect I don't want to have to understand containers or develop tools for them to share this knowledge.

### Tool Writer

1. [x] As a tool writer, I want to get the compatibility spec without pulling the image layer blobs.
1. [x] As a tool writer, I would like to have a library for reading image compatibility so that I can write my own software that takes action based on the spec, e.g.:
   - custom k8s scheduler, admission webhooks, runtime classes etc.
1. [x] As a tool writer, compatibility validation could be integrated into non-runtime tools.
1. [ ] As a tool writer, I want to be able to write tools (that use compatibility specs) that have non-standard applications (e.g., checking individual layers).

### System Runtime Administrator

1. [x] As a system runtime administrator, I want to check whether a container is compatible with the nodes I am going to run it on using the provided tool.
1. [x] As a system runtime administrator, I would like to fetch additional documentation for understanding specific settings in the compatibility spec.
1. [ ] As a system runtime administrator, selecting which image to run should only require pulling the Index manifest, and parsing the descriptors listed.
  Additional API calls to the registry should not be required. (_Image selection by runtimes is out of scope for this proposal_)
1. [x] As a system runtime administrator, I want to validate whether all running applications in the cluster are compatible with a new operating system (or new operating system version) or not before migrating. (_As applications I understand containerized applications, in that case it would be a matter of validating compatibilty against the host_)
1. [x] As a system runtime administrator, I want to validate whether all running applications in the cluster are compatible with new hardware (cpu, gpu, nic etc.) or not before migrating. (_The same note as above_)
1. [ ] As a system runtime administrator, I want to use annotations to schedule for the appropriate resources.
1. [x] As a system runtime administrator, the runtime should not need to know about all possible types of hardware.
   Perhaps hooks could be added for users to inject their own image selection criteria on a given host, or annotations could be injected.

### Deployment Engineer

1. [x] As a deployment engineer, I want to parse an image index and find the “optimal” image for the cluster node I am aiming to run the image on.
That includes being able to:
   - Discover the image that fits the selected host.
   - Find the best match from the nodes and images I have available.
   - Determine that the image is not fitting the selected host.
1. [x] As a deployment engineer, I want the image compatibility check to be performed without downloading or executing the referenced image layers.
1. [x] As a deployment engineer, I should be able to add the compatibility to images already being used in production, especially for the images released before image compatibility wg was created.
1. [x] As a deployment engineer, I want to be able to specify the version and variant of an application or other user specific configuration (e.g. MPI), and not only hardware/kernel details in the compatibility specification.
1. [x] As a deployment engineer, I want my compatibility spec runtime to be able to select the best possible runtime available on a node (e.g. runc vs nvidia vs wasm).
1. [x] As a deployment engineer, I want to be able to rank the images in a multi-platform image so that the runtime can know which one to choose when more than one image is compatible with the runtime environment.
1. [x] As a deployment engineer, I want to reuse community projects so that I don't duplicate and integrate the functionality of the already existing tools.

### Registry Maintainer

1. [x] As a registry user or operator I want to have a common way of inspecting information about image compatibility to enable users to find an image that best matches their system. (_That can be achieved by the tool provided by the OCI group_)
1. [x] As a registry operator (or user that has no control over their registry implementation), I want any compatibility changes to not depend on registry server changes or upgrades.

### OCI Specification Maintainer

1. [x] As a spec maintainer, I want the solution to avoid breaking other specs (confidential images, image signing, existing implementations for runtimes picking images).
1. [x] As a spec maintainer, I don't want the spec to update for new hardware devices, kernel releases, or other external dependencies.
1. [x] As a spec maintainer, I don't want to overlap significantly with solutions from other specs (like SBOMs).

### Security Administrator

1. [x] As a security administrator, I want predictable behavior from runtimes, which does not change based on unsigned content.
1. [x] As a security administrator, I want to ensure the compatibility spec cannot be used to escalate privileges beyond what is requested by the deployment engineer.
1. [x] As a security (or more generally, XYZ) researcher, I want to annotate a container (separately) with my niche jargon of metadata.
1. [x] As a security administrator, I want to know that image compatibility cannot be used to circumvent image signing.
1. [ ] As a security administrator I want to catalog SBOMs of compatible images to put into a report about software used by my group / institution.
