# Proposal H - compatibilities in flat format

The purpose of the proposal is to describe compatibility requirements for container images to run on a host.

This proposal is based on proposal A, B, C, D, E, F and G with new ideas; some description is kept unchanged and borrowed from these original proposals.

The design goal is to avoid over-flexible specification, and avoid abusing the specification.

It is NOT TO store variation of user configuration of applications in the compatibility artifact.

The first part describes the compatibility artifact, including manifest, compatibility spec and examples.

The second part talks about compatibilities embedded inside platform object.

The third part discusses the flexibility of the proposal.

The fourth part compares the pros. and cons. of these two options.

The fifth part discusses how the compatibility spec collaborates with the Node Feature Discovery (NFD) community.

Finally, the last part shows the addressed usage scenarios.

## Compatibility Artifact

### Artifact Manifest

The artifact manifest contains ONLY one blob.

```json
{
  "schemaVersion": "0.1.0",
  "mediaType": "application/vnd.oci.image.manifest.v1+json",
  "artifactType": "application/vnd.oci.image-compatibility.v1",
  "config": {
    "mediaType": "application/vnd.oci.empty.v1+json",
    "digest": "sha256:44136fa355b3678a1146ad16f7e8649e94fb4fc21fe77e8310c060f61caaff8a",
    "size": 2
  },
  "layers": [
    {
      "mediaType": "application/vnd.oci.image-compatibility.spec.v1+json",
      "digest": "sha256:4a47f8ae4c713906618413cb9795824d09eeadf948729e213a1ba11a1e31d052",
      "size": 1710,
      "annotations": {
        "name": "profileH"
      }
    }
  ],
  "subject": {
    "mediaType": "application/vnd.oci.image.manifest.v1+json",
    "digest": "sha256:5b0bcabd1ed22e9fb1310cf6c2dec7cdef19f0ad69efa1f392e94a4333501270",
    "size": 7682
  },
  "annotations": {
    "oci.opencontainers.image.created": "2024-03-27T08:08:08Z"
  }
}
```

The new artifact type for image compatibility MUST be referenced in OCI documentation.

The schema version is to use string semver instead of int version, to help tool in content deserialization.

The media type is defined as vnd.oci.image-compatibility.spec, in order to be defined agnostic to cluster management software.

### Image Compatibility Spec

The spec structure is quite simple, a flat json file.

Schema:

- **`compatibilities`** _list of compatibility sets_

  This REQUIRED property describes all image compatibility sets, contains at least one compatibility set.

  - **`compatibility set`** _list of compatibility requirements_

    This REQUIRED property describes compatibility requirements in one compatibility set.

    If one of the compatibility sets passes the compatibility check against the target node, that means the node is compatible with the image.

    There are three category key-value pair inside the compatibility set.

    - **`compatibility feature field`**

      This REQUIRED property describes a feature compatibility requirement on the target host, the field accepts OCI and community defined labels.

      The value of compatibility field in the compatibility artifact SHOULD be described in contiguous or discontinuous range if the value discovered on nodes can be different.

      It's the tool's responsibility to interpret the value of OCI or community defined feature label, and matching criteria with the value discovered from the node via NFD tool or other tool. NFD means the Kubernetes node feature discovering SIG.

      Inevitably, some compatibility fields MAY be duplicated among compatibility sets.

      The value type of this field is STRING.

    - **`compatibility score field`**

      This OPTIONAL property describes the score of the compatibility set.

      There is only one compatibility score field allowed in one compatibility set.

      The highest score of the compatibility set means this compatibility set is the most optimal one.

      If this field is not present, the score of this compatibility set is 0.

      If compatibility sets have the same score, that means any of these compatibility sets pass the compatibility check is successful in matching the compatibility requirements.

      The compatibility-score is used for selecting optimal image or optimal node in descending order for those compatibility sets pass the compatibility check.

      The value type of this field is INT.

    - **`annotation field`**

      This OPTIONAL property is a field to describe human-readable information regarding to the compatibility set.

      The value type of this field is STRING.

### Artifact example of use

All fields in this section are only for examples.

#### Example - image is compatible with different hardware combinations

```json
{
  "compatibilities": [
    {
      "oci.cpu.vendor": "GenuineIntel",
      "oci.cpu.features": "AVX512FP16,",
      "oci.kernel.selinux": "false",
      "oci.kernel.configurations": "PREEMPT,",
      "oci.kernel.versions": "[4.19, 5.16)",
      "oci.os.glibc": "[2.31, 2.37]",
      "oci.pci.devices": "15B3.020D,"
    },
    {
      "oci.cpu.vendor": "AuthenticAMD",
      "oci.cpu.features": "FPHP,",
      "oci.kernel.selinux": "false",
      "oci.kernel.configurations": "PREEMPT,",
      "oci.kernel.versions": "[4.19, 5.16)",
      "oci.os.glibc": "[2.31, 2.37]",
      "oci.pci.devices": "1002.67ff",
      "annotation": "only work with AMD CPU and AMD GPU"
    }
  ]
}
```

#### Example - image is compatible with optimal hardware

```json
{
  "compatibilities": [
    {
      "oci.cpu.vendor": "GenuineIntel",
      "oci.cpu.features": "AMXFP16,",
      "oci.kernel.selinux": "false",
      "oci.kernel.configurations": "PREEMPT,",
      "oci.kernel.versions": "[4.19, 5.16)",
      "oci.os.glibc": "[2.31, 2.37]",
      "oci.os.openmpi": "[4.1.6, 4.1.6]",
      "oci.pci.devices": "15B3.020D,",
      "compatibility-score": 50
    },
    {
      "oci.cpu.vendor": "GenuineIntel",
      "oci.cpu.features": "AVX512FP16,",
      "oci.kernel.selinux": "false",
      "oci.kernel.configurations": "PREEMPT,",
      "oci.kernel.versions": "[4.19, 5.16)",
      "oci.os.glibc": "[2.31, 2.37]",
      "oci.os.openmpi": "[4.1.6, 4.1.6]",
      "oci.pci.devices": "15B3.020D,",
      "oci.compatibility-score": 100
    },
    {
      "oci.cpu.vendor": "AuthenticAMD",
      "oci.cpu.features": "FPHP,",
      "oci.kernel.selinux": "false",
      "oci.kernel.configurations": "PREEMPT,",
      "oci.kernel.versions": "[4.19, 5.16)",
      "oci.os.glibc": "[2.31, 2.37]",
      "oci.os.openmpi": "[4.1.6, 4.1.6]",
      "oci.pci.devices": "1002.67ff,",
      "oci.compatibility-score": 80,
      "annotation": "only work with AMD CPU and AMD GPU"
    }
  ]
}
```

## Compatibilities Embedded Inside Platform Object

### Extend Platform Object with New Compatibilities Field

Add a new field in platform object in [image index](https://github.com/opencontainers/image-spec/blob/main/image-index.md#image-index-property-descriptions).

The compatibilities author should be aware that the maximal size of image index is 4 MiB.

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

    - **`compatibilities`** _array of string-string maps_

      This new OPTIONAL property describes a list of compatibility sets, contains at least one compatibility set. Same definition as that in Image Compatibility Spec.

### Deprecation Path

The platform.os.version, platform.os.features and platform.features are expected to be moved to platform.compatibilities fields.

The old usage of these fields is deprecated at some point, but that's often reactionary after popular builders stop using these fields.

Backwards compatibility or graceful degradation should be ensured for these fields where possible with either old images, old builders, and/or old runtimes.

It may take years for the deprecation.

## Flexibility for Different Purpose

It's the image author's (or image provider's) freedom to use standalone artifact or platform.compatibilities to express the compatibility requirements, the format will be kept same.

The subject field should be used to associate the compatibility artifact with target image. The referrers API should be used to discover artifacts.

The client will query for an artifact using the referrers' response, selecting the entry with a matching artifactType and the most recent "created" timestamp.

If platform.compatibilities is detected and not empty, then runtime is not required to send referrers API to query compatibility artifact.

If there is no platform.compatibilities, or the content of platform.compatibilities is empty, it should be configurable whether to send referrers API to registry to query compatibility artifact.

## Pros. and Cons. Comparison

### Pros. and Cons. of Compatibility Artifact

It's possible to update compatibility independently without having to re-release and re-distribute image.

In some vertical industries, re-distributing image means repeating release cycle to production site, it may take several weeks to several months for complex applications, very time-consuming and high cost.

The application using new image has to pass series tests inside vendor's lab and then customer's lab, and then roll out in production site gradually. Tests may include function, performance, reliability, security, inter-working etc.

The cons. of compatibility artifact is, if it's used for image selection or scheduling decision, additional referrers API request should be sent to registry.

When the compatibility artifact is used for validation purpose, it can be processed without runtime image selection or scheduling decision being involved.

### Pros. and Cons. of platform.compatibilities

Obviously current API interaction with registry can retrieve the compatibility requirements directly, no additional referrers API request is needed.

Modification to the platform.compatibilities will alter the image index content.

Image shipping will usually try to ensure the integrity of the image, and prevent unauthorized tampering, thus image index content update will require re-distribute the image for supply chain security.

## Compatibility and NFD filed labels

The proposal itself does not intend to define concrete compatibility field labels, it should be a task for separate proposals.

To free the tools' implementation, and make sure the existing os.version, os.features, platform.features can be smoothly deprecated, and reduce the sync. effort between OCI and NFD, it's reasonable just to define minimal set of OCI compatibility fields.

These field labels could be easily mapped or translated to NFD field labels.

Because the value of compatibility field SHOULD be described in contiguous or discontinuous range if the value discovered on nodes can be different, NFD labels are dynamically created according to node features, it's not one to one mapping.

For example, oci.kernel.configurations may contain several boot configurations in a single string, but in NFD, several labels like kernel-config.PREEMPT, kernel-config.intel_iommu may be applied.

If the field is required in image compatibilities, but does not present in NFD labels, the new feature label should be discussed and contributed into NFD.

It's expected that NFD feature labels definition and node feature discovering could be an independent component in NFD, and could be consumed by different consumers.

To avoid over-flexible specification, and avoid abusing the specification, a way to limit the freedom is needed.

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
