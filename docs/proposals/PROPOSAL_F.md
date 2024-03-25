# Proposal F - Artifact Compatibility Outline

> Maintained by Compatibility Communities

This is a slimmed down, simpler version of [Proposal D](PROPOSAL_D.md). It is intended to provide the skeleton structure to advise artifact generation for groups that want to develop artifacts for compatibilility without enforcing strong requirements for the content of the attributes.  We still define the following definitions:

1. A **compatibility spec** is defined by a compatibility interest group, and defines a known set of metadata attributes and relationships between them. There would be one copy of this, in a standard location, one source of truth (versioned).
1. A **compatibility interest group** is a community that maintains an artifact definition.
1. An extraction of metadata attributes defined by a compatibility interest group, and for a specific container image or application is a **compatibility spec**. The degree to which the attributes are namespaced, organized, and otherwise defined is up to the compatibility interest group.

## Use Cases

Compatibility metadata, and specifically external artifacts that are parsed by different plugins intended for diverse environments, can satisfy many use cases. The following are examples.

- **scheduling**: Schedulers need to see image compatibility metadata to make decisions. Having the metadata only available in the image manifest (at the image selection or pulling phase) does not satisfy this use case, which needs to retrieve metadata for one or more images apriori. The external artifact structure works well here, as scheduling communities could easily use external artifacts with external plugins to inform their schedulers. In HPC the example use case is Flux Framework, a graph-based scheduler that would define different resource subsystems in a graph (for image compatibility metadata to match to) and in Kubernetes an example use case is Node Feature Discovery (NFD) that would warrant wanting their annotations ascribed to images.

- **image selection**: is a use case only in that the tool selecting the image does not eventually adopt one of the simpler proposals [A](PROPOSAL_A.md) or [E](PROPOSAL_E.md), or has use case that is not addressed by a traditional container runtime pulling the image (and using image manifest metadata).

- **provenance**: a provenance tool can easily keep a database of image metadata based on artifacts alone to curate images available on a system, or similar use cases that require such a database.

### Benefits

- A compatibility interest group is free to namespace and define whatever metadata annotations their tooling needs. For example, if a graph schema is required to describe relationships between attributes, the reference URL can be a known metadata attribute. If documentation is desired, this is possible too.
- The simple structure supports the most basic of use cases (key/value pair checking) to more advanced (graph-based scheduling).

## Image Compatibility Artifact

In [Proposal B](PROPOSAL_B.md) it is stated that a compatibility artifact should describe CRITICAL requirements for container images to run on a host, which is inferred to mean a limited set that determines compatibility. This proposal takes the stance that we don't yet know what that minimal set is, and further, that it may vary depending on the use case for any specific artifact. Instead of declaring a minimum set, this proposal states that the artifact should contain minimum requirements for compatibility but can contain more that MAY be used by plugins to not just determine compatibility but optimal usage. The artifact serves as a source of metadata about container requirements, and it is on the level of the plugins or associated tooling to decide which subset should be used for a given use case. Arguably, this set will change. More discussion and research is required to understand what the line between critical (for the image to run, compatibility) and running optimally.

### Artifact Manifest

This artifact MAY have a manifest that references it with the media type that is specific to the compatibility interest group. For example, metadata for node feature discovery may take the following form:

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
      "mediaType": "application/k8s.nfd.image-compatibility.spec.v1+json",
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

The artifact is a simple json structure of named groups and within them, key value pairs. The groups can provide a higher level organization, such as types of features or even versions. This organization is up to the compatibility interest group. Here is an example organizing NFD annotations within extractor groups (a random selection):

```json
{
  "compatibilities": {
    "cpu": {
      "cpu-hardware_multithreading": true,
      "cpu-pstate.status": "passive",
      "cpu-pstate.turbo": false
    },
    "kernel": {
      "kernel-selinux.enabled": true,
      "kernel-version.major": 4           
    },
    "memory": {
      "memory-numa": true,
      "memory-nv.dax": true
    },
    "network": {
      "network-sriov.capable": true,
      "network-sriov.configured": true
    }
}
```

A similar spec for an HPC-oriented container might look like the following:

```json
{
  "compatibilities": {
    "cpu": {
      "target": "arm64",
    },
    "mpi": {
      "implementation": "openmpi",
      "version": "4.1.6"           
    },
    "gpu": {
      "enabled": true
    },
    "os": {
      "vendor": "rocky",
      "version": "8"
    }
}
```

As stated previously, the granularity of metadata provided is up to the compatibility interest group. The degree to which each metadata attribute is used is up to the plugin or tool within the context of a specific use case.

## References

- [Original planning document](https://docs.google.com/document/d/1nRUuW9i7NRdYXrUr8DQXLMETXwP1J9rw3TbpLTU9yho/edit)
- [Use Cases](https://docs.google.com/document/d/1ULXHY0pdiLBlGZPN2Gj54XSvAo--UXKYr4opTrWSFMs/edit)
- [Graph prototype added here](https://github.com/supercontainers/compspec/pull/6)
