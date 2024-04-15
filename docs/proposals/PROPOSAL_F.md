# Proposal F - Artifact Compatibility Outline

> Maintained by Compatibility Communities

This is a slimmed down, simpler version of [Proposal D](PROPOSAL_D.md). It is intended to provide the skeleton structure to advise artifact generation for groups that want to develop artifacts for compatibilility without enforcing strong requirements for the content of the attributes.  We still define the following definitions:

1. A **compatibility spec** is defined by a compatibility interest group, and defines a known set of metadata attributes and relationships between them. There would be one copy of this, in a standard location, one source of truth (versioned).
1. A **compatibility interest group** is a community that maintains an artifact definition.
1. An extraction of metadata attributes defined by a compatibility interest group, and for a specific container image or application is a **compatibility spec**. The degree to which the attributes are namespaced, organized, and otherwise defined is up to the compatibility interest group.

## Use Cases

Compatibility metadata, and specifically external artifacts that are parsed by different plugins intended for diverse environments, can satisfy many use cases. The following are examples.

- **scheduling**: Schedulers need to see image compatibility metadata to make decisions. Having the metadata only available in the image manifest (controlled by way of OCI) is severely limiting to niche communities that might need additional (less common) information for scheduling. An example comes from the HPC community with MPI.

- **image selection**: is a use case only in that the tool selecting the image does not eventually adopt one of the simpler proposals [A](PROPOSAL_A.md) or [E](PROPOSAL_E.md), or has use case that is not addressed by a traditional container runtime pulling the image (and using image manifest metadata).

- **provenance**: a provenance tool can easily keep a database of image metadata based on artifacts alone to curate images available on a system, or similar use cases that require such a database.

### Benefits

- A compatibility interest group is free to namespace and define whatever metadata annotations their tooling needs. For example, if a graph schema is required to describe relationships between attributes, the reference URL can be a known metadata attribute. If documentation is desired, this is possible too.
- The simple structure supports the most basic of use cases (key/value pair checking) to more advanced (graph-based scheduling).

## Image Compatibility Artifact

In [Proposal B](PROPOSAL_B.md) it is stated that a compatibility artifact should describe CRITICAL requirements for container images to run on a host, which is inferred to mean a limited set that determines compatibility. This proposal takes the stance that we don't yet know what that minimal set is, and further, that it may vary depending on the use case for any specific artifact. Instead of declaring a minimum set, this proposal states that the artifact should contain minimum requirements for compatibility but can contain more that MAY be used by plugins to not just determine compatibility but optimal usage. The artifact serves as a source of metadata about container requirements, and it is on the level of the plugins or associated tooling to decide which subset should be used for a given use case. Arguably, this set will change. More discussion and research is required to understand what the line between critical (for the image to run, compatibility) and running optimally.

### Artifact Manifest

This artifact MAY have a manifest that references it with the compatibility artifact media type. For example:

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
      "mediaType": "application/vnd.oci.image-compatibility.spec.v1+json",
      "digest": "sha256:4a47f8ae4c713906618413cb9795824d09eeadf948729e213a1ba11a1e31d052",
      "size": 1710
    }
  ],
  "annotations": {
    "oci.opencontainers.image.created": "2024-01-02T03:04:05Z"
  }
}
```

### Artifact

The artifact is a simple json structure under the top level key "compatibilities" of named groups and within them, key value pairs, where it is strongly suggested that each top level key refers to a compatibility group. Each compatibility special interest group that maintains an underlying namespace can choose the specific key value pairs within. Here is an example with NFD annotations. Note that the specific identifier for NFD is likely more complex than just "nfd" (e.g., io.kubernetes.nfd):

```json
{
  "compatibilities": {
    "nfd": {
      "cpu-hardware_multithreading": true,
      "cpu-pstate.status": "passive",
      "cpu-pstate.turbo": false
      "kernel-selinux.enabled": true,
      "kernel-version.major": 4           
      "memory-numa": true,
      "memory-nv.dax": true
      "network-sriov.capable": true,
      "network-sriov.configured": true
    }
}
```

The artifact above shows just one section. Artifacts can include as many sections as desired.
A similar spec for an HPC-oriented container might look like the following:

```json
{
  "compatibilities": {
    "io.archspec": {
      "target": "arm64",
    },
    "org.supercontainers": {
      "mpi.implementation": "openmpi",
      "mpi.version": "4.1.6"           
    }
}
```

As stated previously, the granularity of metadata provided is up to the compatibility interest group. The degree to which each metadata attribute is used is up to the plugin or tool within the context of a specific use case.

## Requirements

Taken from [REQUIREMENTS](../REQUIREMENTS.md). Points that have:

- `TOOL RESPONSIBILITY` would be possible by a custom tool.
- `AGNOSTIC`: the proposal is agnostic to the specific point (can be decided by working group)

### Image Author

- [x] As an image author, I want to update compatibility independently without having to re-release and re-distribute my image.
- [x] As an image author, I want to have the freedom to express any compatibility that is necessary for my container to run on the host.
  - [x] Including conditional compatibility, such as a container running on AMD CPU and intel CPU having different requirements.
  - [x] Including must have/nice to have compatibility.
  - [x]  Matching finer grain platform definitions.
- [x] As an image author, I want to use a provided tool to verify the compatibility spec I wrote against the schema.
- [ ] As an image author, I would like to create a single compatibility description that is common to a group of images (`TOOL RESPONSIBILITY`).
- [ ] As an image author, I want to be able to specify in my manifest that my multi-platform image has a compatibility spec which should be consulted `AGNOSTIC`.
- [ ]  As an image author, I want to be able to include compatibility specifications from base layers that I inherited from `TOOL RESPONSIBILITY` that can derive path of base layers
- [x] As an image author, I want to ensure that runtimes without image compatibility gracefully fall back to running a usable image.

### Domain Architect

- [x] As a domain architect I want a process to share my knowledge about \<niche topic\> compatibility with some compatibility interest group.
- [x] As a domain architect I don't want to have to understand containers or develop tools for them to share this knowledge.

### Tool Writer

- [x] As a tool writer, I want to get the compatibility spec without pulling the image layer blobs.
- [x] As a tool writer, I would like to have a library for reading image compatibility so that I can write my own software that takes action based on the spec, e.g.:
  - [x] custom k8s scheduler, admission webhooks, runtime classes etc.
- [x] As a tool writer, compatibility validation could be integrated into non-runtime tools.
- [x] As a tool writer, I want to be able to write tools (that use compatibility specs) that have non-standard applications (e.g., checking individual layers).

### System Runtime Administrator

- [x] As a system runtime administrator, I want to check whether a container is compatible with the nodes I am going to run it on using the provided tool.
- [x] As a system runtime administrator, I would like to fetch additional documentation for understanding specific settings in the compatibility spec.
- [ ] As a system runtime administrator, selecting which image to run should only require pulling the Index manifest, and parsing the descriptors listed.
  Additional API calls to the registry should not be required.
- [ ] As a system runtime administrator, I want to validate whether all running applications in the cluster are compatible with a new operating system (or new operating system version) or not before migrating. `TOOL RESPONSIBILITY`
- [ ] As a system runtime administrator, I want to validate whether all running applications in the cluster are compatible with new hardware (cpu, gpu, nic etc.) or not before migrating `TOOL RESPONSIBILITY`.
- [ ] As a system runtime administrator, I want to use annotations to schedule for the appropriate resources (we do not have annotations, we have metadata attributes in the artifact).
- [x] As a system runtime administrator, the runtime should not need to know about all possible types of hardware.
- [ ] Perhaps hooks could be added for users to inject their own image selection criteria on a given host, or annotations could be injected `TOOL RESPONSIBILITY`.

### Deployment Engineer

- [ ] As a deployment engineer, I want to parse an image index and find the “optimal” image for the cluster node I am aiming to run the image on `AGNOSTIC`.
That includes being able to:
  - Discover the image that fits the selected host.
  - Find the best match from the nodes and images I have available.
  - Determine that the image is not fitting the selected host.
- [x] As a deployment engineer, I want the image compatibility check to be performed without downloading or executing the referenced image layers.
- [x] As a deployment engineer, I should be able to add the compatibility to images already being used in production, especially for the images released before image compatibility wg was created.
- [x] As a deployment engineer, I want to be able to specify the version and variant of an application or other user specific configuration (e.g. MPI), and not only hardware/kernel details in the compatibility specification.
- [ ] As a deployment engineer, I want my compatibility spec runtime to be able to select the best possible runtime available on a node (e.g. runc vs nvidia vs wasm) `TOOL RESPONSIBILITY`
- [x] As a deployment engineer, I want to be able to rank the images in a multi-platform image so that the runtime can know which one to choose when more than one image is compatible with the runtime environment.
- [x] As a deployment engineer, I want to reuse community projects so that I don't duplicate and integrate the functionality of the already existing tools.

### Registry Maintainer

- [x] As a registry user or operator I want to have a common way of inspecting information about image compatibility to enable users to find an image that best matches their system.
- [x] As a registry operator (or user that has no control over their registry implementation), I want any compatibility changes to not depend on registry server changes or upgrades.

### OCI Specification Maintainer

- [x] As a spec maintainer, I want the solution to avoid breaking other specs (confidential images, image signing, existing implementations for runtimes picking images).
- [x] As a spec maintainer, I don't want the spec to update for new hardware devices, kernel releases, or other external dependencies.
- [x] As a spec maintainer, I don't want to overlap significantly with solutions from other specs (like SBOMs).

### Security Administrator

- [x] As a security administrator, I want predictable behavior from runtimes, which does not change based on unsigned content. _(runtimes should ignore unsigned or untrusted artifacts if signed images are required, even if the image itself is signed by a trusted authority)_
- [x] As a security administrator, I want to ensure the compatibility spec cannot be used to escalate privileges beyond what is requested by the deployment engineer.
- [x] As a security (or more generally, XYZ) researcher, I want to annotate a container (separately) with my niche jargon of metadata.
- [x] As a security administrator, I want to know that image compatibility cannot be used to circumvent image signing.
- [ ] As a security administrator I want to catalog SBOMs of compatible images to put into a report about software used by my group / institution `TOOL RESPONSIBILITY`

## References

- [Original planning document](https://docs.google.com/document/d/1nRUuW9i7NRdYXrUr8DQXLMETXwP1J9rw3TbpLTU9yho/edit)
- [Use Cases](https://docs.google.com/document/d/1ULXHY0pdiLBlGZPN2Gj54XSvAo--UXKYr4opTrWSFMs/edit)
- [Graph prototype added here](https://github.com/supercontainers/compspec/pull/6)
