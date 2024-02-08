# Requirements

This document contains a list of requirements identified
to be considered in all proposals originating from this WG.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" are to be interpreted as described in [RFC 2119](https://tools.ietf.org/html/rfc2119) (Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997).

## Definitions

- **Image Compatibility specification**: A specification that describes container image requirements for a host operating system.

## Roles

The following is a list of roles used to describe different contexts from a user story perspective:

- **Image Author**: provides the image artifact and maintains container with compatibility specification.
- **Domain Architect**: provides the expertise for how images should be described in the compatibility specification.
- **Tool Writer**: builds tools to manage images on registries and other locations.
- **System Runtime Administrator**: maintains the container runtime environment on which OCI containers run.
- **Deployment Engineer**: manages the application deployment on to a container runtime, cluster or host.
- **Registry Maintainer**: maintains the registry (e.g., Docker Hub, Harbor).
- **OCI Specification Maintainer**: maintains the open containers specification (aka us).
- **Security Administrator**: maintains security aspects of environments.

## User Stories

The following are user stories grouped by role.

### Image Author

1. As an image author, I want to update compatibility independently without having to re-release and re-distribute my image.
1. As an image author, I want to have the freedom to express any compatibility that is necessary for my container to run on the host.
   - Including conditional compatibility, such as a container running on AMD CPU and intel CPU having different requirements.
   - Including must have/nice to have compatibility.
   - Matching finer grain platform definitions.
1. As an image author, I want to use a provided tool to verify the compatibility spec I wrote against the schema.
1. As an image author, I would like to create a single compatibility description that is common to a group of images.
1. As an image author, I want to be able to specify in my manifest that my multi-platform image has a compatibility spec which should be consulted.
1. As an image author, I want to be able to include compatibility specifications from base layers that I inherited from.
1. As an image author, I want to ensure that runtimes without image compatibility gracefully fall back to running a usable image.

### Domain Architect

1. As a domain architect I want a process to share my knowledge about \<niche topic\> compatibility with some compatibility interest group.
1. As a domain architect I don't want to have to understand containers or develop tools for them to share this knowledge.

### Tool Writer

1. As a tool writer, I want to get the compatibility spec without pulling the image layer blobs.
1. As a tool writer, I would like to have a library for reading image compatibility so that I can write my own software that takes action based on the spec, e.g.:
   - custom k8s scheduler, admission webhooks, runtime classes etc.
1. As a tool writer, compatibility validation could be integrated into non-runtime tools.
1. As a tool writer, I want to be able to write tools (that use compatibility specs) that have non-standard applications (e.g., checking individual layers).

### System Runtime Administrator

1. As a system runtime administrator, I want to check whether a container is compatible with the nodes I am going to run it on using the provided tool.
1. As a system runtime administrator, I would like to fetch additional documentation for understanding specific settings in the compatibility spec.
1. As a system runtime administrator, selecting which image to run should only require pulling the Index manifest, and parsing the descriptors listed.
  Additional API calls to the registry should not be required.
1. As a system runtime administrator, I want to validate whether all running applications in the cluster are compatible with a new operating system (or new operating system version) or not before migrating.
1. As a system runtime administrator, I want to validate whether all running applications in the cluster are compatible with new hardware (cpu, gpu, nic etc.) or not before migrating.
1. As a system runtime administrator, I want to use annotations to schedule for the appropriate resources.
1. As a system runtime administrator, the runtime should not need to know about all possible types of hardware.
   Perhaps hooks could be added for users to inject their own image selection criteria on a given host, or annotations could be injected.

### Deployment Engineer

1. As a deployment engineer, I want to parse an image index and find the “optimal” image for the cluster node I am aiming to run the image on.
That includes being able to:
   - Discover the image that fits the selected host.
   - Find the best match from the nodes and images I have available.
   - Determine that the image is not fitting the selected host.
1. As a deployment engineer, I want the image compatibility check to be performed without downloading or executing the referenced image layers.
1. As a deployment engineer, I should be able to add the compatibility to images already being used in production, especially for the images released before image compatibility wg was created.
1. As a deployment engineer, I want to be able to specify the version and variant of an application or other user specific configuration (e.g. MPI), and not only hardware/kernel details in the compatibility specification.
1. As a deployment engineer, I want my compatibility spec runtime to be able to select the best possible runtime available on a node (e.g. runc vs nvidia vs wasm).
1. As a deployment engineer, I want to be able to rank the images in a multi-platform image so that the runtime can know which one to choose when more than one image is compatible with the runtime environment.
1. As a deployment engineer, I want to reuse community projects so that I don't duplicate and integrate the functionality of the already existing tools.

### Registry Maintainer

1. As a registry user or operator I want to have a common way of inspecting information about image compatibility to enable users to find an image that best matches their system.
1. As a registry operator (or user that has no control over their registry implementation), I want any compatibility changes to not depend on registry server changes or upgrades.

### OCI Specification Maintainer

1. As a spec maintainer, I want the solution to avoid breaking other specs (confidential images, image signing, existing implementations for runtimes picking images).
1. As a spec maintainer, I don't want the spec to update for new hardware devices, kernel releases, or other external dependencies.
1. As a spec maintainer, I don't want to overlap significantly with solutions from other specs (like SBOMs).

### Security Administrator

1. As a security administrator, I want predictable behavior from runtimes, which does not change based on unsigned content.
1. As a security administrator, I want to ensure the compatibility spec cannot be used to escalate privileges beyond what is requested by the deployment engineer.
1. As a security (or more generally, XYZ) researcher, I want to annotate a container (separately) with my niche jargon of metadata.
1. As a security administrator, I want to know that image compatibility cannot be used to circumvent image signing.
1. As a security administrator I want to catalog SBOMs of compatible images to put into a report about software used by my group / institution.
