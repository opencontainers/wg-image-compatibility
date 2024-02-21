# Proposal D - Namespaced and Graph-based Compatibility Metadata Schema

> Also maintained by Compatibility Communities

Proposal D provides a simple means to represent metadata attributes about compatibility in a **compatibility spec** and an optional way to further describe relationships (edges) between attributes (nodes) for those metadata attributes. The latter uses the [JSON Graph format](https://jsongraphformat.info/) (JGF), an already existing standard, to map these attributes into a graph space. In summary, there are three things here:

1. A **compatibility schema** is defined by a compatibility interest group, and  defines a known set of metadata attributes and relationships between them. There would be one copy of this, in a standard location, one source of truth (versioned).
2. An extraction of metadata attributes for one or more compatibility schemas, and for a specific container image or application is a **compatibility spec**
3. The schema is validated by the JSON Graph format (V2) while the spec is validated by [a custom schema that we provide](https://github.com/supercontainers/compspec/blob/main/schema.json)

Note that this proposal has a matching prototype at [compspec-go](https://github.com/compspec/compspec.go) for which full [experiments have been run](https://github.com/rse-ops/lammps-matrix/blob/main/experiment/flux-operator/img/amd64/lammps-reasons-failure_lammps-reasons-failure.png) and a scheduler use case is under development.

## Why do we want a graph?

There are two use cases I'm currently considering for these compatibility metadata:

- **image selection**: is the primary use case. At face value a graph might not seem valuable, because an algorithm could very simply (in a brute force manner) match a set of attributes to a desired set. And to be clear, that use case is not out of scope - the metadata attributes are provided in the compatibility spec in an easy to read and parse, flat format. However, given that we have these terms map into a graph defined by the schema, we can optimize the image selection process by pruning off entire subtrees (that are impossible, for example) before we even consider more detailed metadata. A scaled image selection task would be more computationally feasible than if using a brute force matching approach. Such an example might be to ask for higher level queries across all images in a registry.

- **scheduling**: Schedulers can often represent resources as graphs (e.g., Flux Framework), and this community would want to be able to easily map one or more compatibility schemas into their scheduler graphs.

The design suggested in this proposal addresses both use cases above.

### Benefits

Several benefits to this design include:

- Parsing (for scheduling or image selection) means we can generate one graph (and don't need to combine graphs)
- We don't need to handle cases of "single node" graphs (metadata attributes that are nodes but don't have edges)
- We don't need to keep around an extra jsonschema to validate a custom format, we use a common, community standard (JGF)
- Simple cases of image selection do not need to consider the graph and can use key value pairs provided in the compatibility spec
- Changes to the graph (nodes or edges) are in one place (one source of truth) and versioned, so any upstream change does not require updating artifacts.

## Definitions

- A **Compatibility Interest Group** (CIG) has a vested interest in defining compatibility.
  - Defines namespace for their community metadata
  - Maintains a schema "compspec.json" in a web-accessible location for their community metadata
  - Optionally maintains plugins or associated tooling
- A **Compatibility Schema** is a set of namespaced metadata that describes a compatibiilty interest group's space of expertise.
  - Maintained by a compatibility interest group
  - Metadata is described in a graph format, [Json Graph Format](https://github.com/jsongraph/json-graph-specification), version 2.0, with edges and nodes required.
  - The graph id is the namespace or label for their community metadata (e.g,. `supercontainers.org` or `archspec.io`)
  - Every node must be represented in the graph (defined in edges), and there should be common validation for this.
  - Each nodes is a specific metadata attribute (e.g., `cpu` and `cpu.family`)
  - Each edge is a relationship between two nodes (e.g., `cpu` contains -> `cpu.family`.
- A **Compatibility Specification** is compatibility metadata that describes a container image or application
  - Includes one or more compatibility schemas
  - Also is validated by way of links to the compspec.json for each

Importantly, there is no complex graph structure in artifact itself, but rather in the single, top level schema that defines the space of metadata attributes and relationships. The tools can use the graph (nodes and edges) as needed for their use case. This means they _are not required_ to use them and can simply view the compatibility spec (what we might refer to as an artifact) as a listing of metadata attributes. This gives power and freedom to tool developers to create tools as they see fit. Several designs are afforded:

1. **Simple checking of key/value pairs**: A community might have an algorithm that is as simple as their tool checking for particular keys, and that values match for a host of interest.
2. **Graph-based logic**: A community might warrant using advanced graph parsing to do a similar assessment. An example comes from [Archspec](https://github.com/archspec), where a single architecture (from a flat artifact or the platform, depending on what the tool chooses) can [be placed in a graph](https://github.com/archspec/archspec-json/blob/master/cpu/microarchitectures.json) and used for advanced assessment of compatibility.

## Modifications

The compatibility schema that defines the structure of the artifacts and relationships between attributes would need to live in a single, known location, a web accessible URL that would be maintained by the compatibility interest group, and would be versioned. The artifacts would reference the version they are using. The compatibility artifact JSON blob would live alongside images in a registry.

## Compatibility Schema

The compatibility schema is a single source of truth (with versions) that lives in a web accessible location and defines, for a compatibility interest group:

1. metadata attributes that are valid (node)
2. relationships between attributes (edges)

All nodes must be defined in the graph, and there should be a validation tool for ensuring this. Likely OCI can provide a template repository that has this validation and structure ready to go, and a compatibility interest group simply needs to fill in their metadata attribute names and relationships.

### Example

Note that examples are coming from [supercontainers/compspec](https://github.com/supercontainers/compspec).
Here is an example compatibility schema that defines a set of nodes (metadata attributes) and relationships between them (edges, or how they connect to form a graph):

```json
{
  "graph": {
    "id": "org.supercontainers",
    "type": "compspec",
    "label": "compatibilities",
    "nodes": {
      "mpi": {
        "label": "message passing interface (mpi)"
      },
      "mpi.implementation": {
        "label": "mpi implementation"
      },
      "mpi.version": {
        "label": "mpi version"
      },
      "mpi.portability": {
        "label": "mpi portability attributes"
      },
      "mpi.portability.optimization": {
        "label": "mpi portability optimization"
      },
      "mpi.portability.mode": {
        "label": "mpi portability mode"
      },
      "os": {
        "label": "operating system"
      },
      "os.name": {
        "label": "operating system name"
      },
      "os.release": {
        "label": "operating system release"
      },
      "os.vendor": {
        "label": "operating system vendor"
      },
      "os.version": {
        "label": "operating system version"
      },
      "hardware": {
        "label": "hardware"
      },
      "hardware.gpu": {
        "label": "gpu hardware"
      },
      "hardware.gpu.enabled": {
        "label": "true if gpu is enabled"
      },
      "hardware.gpu.driver": {
        "label": "gpu driver"
      },
      "hardware.gpu.architecture": {
        "label": "gpu architecture"
      },
      "hardware.gpu.driver.version": {
        "label": "gpu driver version"
      },
      "hardware.gpu.cuda": {
        "label": "gpu cuda"
      },
      "hardware.gpu.cuda.version": {
        "label": "cuda version"
      },
      "communication": {
        "label": "communication"
      },
      "communication.framework": {
        "label": "communication framework"
      }
    },
    "edges": [
      {
        "source": "mpi",
        "target": "mpi.version",
        "relation": "contains"
      },
      {
        "source": "mpi",
        "target": "mpi.implementation",
        "relation": "contains"
      },
      {
        "source": "mpi",
        "target": "mpi.portability",
        "relation": "contains"
      },
      {
        "source": "mpi.portability",
        "target": "mpi.portability.optimization",
        "relation": "contains"
      },
      {
        "source": "mpi.portability",
        "target": "mpi.portability.mode",
        "relation": "contains"
      },
      {
        "source": "os",
        "target": "os.name",
        "relation": "contains"
      },
      {
        "source": "os",
        "target": "os.release",
        "relation": "contains"
      },
      {
        "source": "os",
        "target": "os.vendor",
        "relation": "contains"
      },
      {
        "source": "os",
        "target": "os.version",
        "relation": "contains"
      },
      {
        "source": "hardware",
        "target": "hardware.gpu",
        "relation": "contains"
      },
      {
        "source": "hardware.gpu",
        "target": "hardware.gpu.enabled",
        "relation": "contains"
      },
      {
        "source": "hardware.gpu",
        "target": "hardware.gpu.driver",
        "relation": "contains"
      },
      {
        "source": "hardware.gpu.driver",
        "target": "hardware.gpu.driver.version",
        "relation": "contains"
      },
      {
        "source": "hardware.gpu",
        "target": "hardware.gpu.architecture",
        "relation": "contains"
      },
      {
        "source": "hardware.gpu",
        "target": "hardware.gpu.version",
        "relation": "contains"
      },
      {
        "source": "hardware.gpu",
        "target": "hardware.gpu.cuda",
        "relation": "contains"
      },
      {
        "source": "hardware.gpu.cuda",
        "target": "hardware.gpu.cuda.version",
        "relation": "contains"
      },
      {
        "source": "communication",
        "target": "communication.framework",
        "relation": "contains"
      }
    ],
    "metadata": {
      "version": "0.0.0",
      "source": "https://github.com/supercontainers/compspec"
    }
  }
}
```

## Compatibility Artifact

The compatibility artifact would include a subset of metadata attributes that are known to a schema, along with versions and links to the schemas used.

### Example Compatibility Artifact

From the example above we might derive the following compatibility artifact.

```json
{
  "version": "0.0.0",
  "kind": "CompatibilitySpec",
  "metadata": {
    "name": "lammps-prototype",
    "jsonSchema": "https://raw.githubusercontent.com/supercontainers/compspec/main/supercontainers/compspec.json"
  },
  "compatibilities": [
    {
      "name": "org.supercontainers",
      "version": "0.0.0",
      "annotations": {
        "mpi.implementation": "mpich",
        "mpi.version": "4.1.1",
        "os.name": "Ubuntu 22.04.3 LTS",
        "os.release": "22.04.3",
        "os.vendor": "ubuntu",
        "os.version": "22.04",
        "hardware.gpu.available": "yes"
      }
    }
  ]
}
```

Note that we could easily add another compatibility interest group namespace, e.g, [archspec.io](https://github.com/supercontainers/compspec/blob/main/archspec/compspec.json):

```json
{
  "version": "0.0.0",
  "kind": "CompatibilitySpec",
  "metadata": {
    "name": "lammps-prototype"
  },
  "compatibilities": [
    {
      "name": "io.archspec",
      "version": "0.0.0",
      "schema": "https://raw.githubusercontent.com/supercontainers/compspec/main/archspec/compspec.json",
      "annotations": {
        "cpu.model": "13th Gen Intel(R) Core(TM) i5-1335U",
        "cpu.target": "amd64",
        "cpu.vendor": "GenuineIntel"
      }
    },
    {
      "name": "org.supercontainers",
      "version": "0.0.0",
      "schema": "https://raw.githubusercontent.com/supercontainers/compspec/main/supercontainers/compspec.json",
      "annotations": {
        "mpi.implementation": "mpich",
        "mpi.version": "4.1.1",
        "os.name": "Ubuntu 22.04.3 LTS",
        "os.release": "22.04.3",
        "os.vendor": "ubuntu",
        "os.version": "22.04",
        "hardware.gpu.available": "yes"
      }
    }
  ]
}
```

## Frequently Asked Questions

### How would graphs be combined and parsed?

Graphs would not be combined. The goal with the design above would be to generate one graph that includes the entire namespace.
E.g., in the example below, we see that the two schemas are represented side by side, with a common root of the "image" or application.

```console
                                +--------+
                                | image  |
                            +---+--------+------+
                            |                   |
                            |                   |
                      +-----v------+  +---------v----------+
                      |archspec.io |  |supercontainers.org |
                      +------------+ ++------+--------+----+
                       |             |       |        |
                    +--v--+       +--v--+ +--v-+ +----v-----+
                    | cpu |       | mpi | | os | | hardware |
    +-----------+---+-----+----+  +-----+ +----+ +----------+
    |           |              |     v      v          v
    |           |              |
+---v---+   +---v----+     +---v----+
| model |   | target |     | vendor |
+-------+   +--------+     +--------+
```

The above is abstract in that we snipped the tree for supercontainers.org at the top level (mpi, os, hardware) only because there were a lot of children. It is also abstract because we are showing labels instead of the actual values (e.g., a cpu model is a specific set of values) and at each node we can imagine an entire set of matching container images. While describing algorithms to build or traverse a graph is outside of scope of this proposal, a simple approach would be to:

1. Construct the graph from the compspec.json for all relevant schemas (those defined for a set of compatibility specs of interest)
2. Map the specific compatibility specs to it, and ensure that every node has a link to an entire image (we do this so we can prune the graph at any point and not need to traverse into children to find what children are that (at leaves)
3. Do a depth first search to find the first match for a compatibility request (a set of metadata attributes that need to be matched)
4. A greedy approach would stop and return the first match.

Importantly, because there are not dependencies between graphs, this means a subtree per compatibility schema. This allows for two schemas to define similar attributes without any risk of conflict.

### Why do we need a graph?

Arguably, a metadata attribute like `mpi.implementation` already has a nested hierarchy implicit in the name. The relationship implied here is one of a child and parent, or "contains." However, there are relationships (edges) between metadata attributes that often don't adhere to this. A set of examples could be power, network, and IO patterns. For describing compatibility, we might want to say something like "node A draws from / supplies to node types B...C...D." Aside from those two being in separate namespaces (e.g., power describing physical resources) we likely are going to have the same kind of edge (relationship) describing more than one thing. It's not just a flat 1:1 "this node A contains B" kind of thing. For IO, we might have a graph that describes patterns and we might want to say "this container is going to be heavy for this particular read pattern and should be used with these kinds of filesystems." That is also not a traditional parent/child or 1:1 relationship. Finally, network is extremely important for compatibility. Most cloud networks will simply not have the low latency that is required for an MPI application. We would need to be able to have metadata attributes that describe the network and minimum (or range) of values that are acceptable for the application to run. The failure case would be pulling a large container to 128 nodes and then discovering it doesn't even start because of the network (a common occurrence for many of our applications and the current tendency to maximize bandwidth over anything else for cloud).

### How are OR / AND preferences requested by the user?

The compatibility spec itself does not include a graph, and this is an explicit decision to keep the artifact as simple as possible, and (given a change to relationships) to require the change in just one place (the schema) and not across every artifact that might define it. However, there are several alternatives that could be considered to serve this same functionality.

- For `AND` this is a simple case of requesting two metadata attributes.
- For ranges (e.g., `MIN` and `MAX` there might be a single value provided, and a plugin or parsing tool would allow for specification of that logic to check against the single metadata value label.
- If the `OR` is a truth (e.g., value A is the same as value B) and the user is asking for "A OR B" then this information would be represented in the compatibility schema graph as an `IS` The schema would it's the same as another, and would not place the burden on every user (writing a compatibility spec) to know that.
- If it's not a truth, and you want an OR, this is on the level of the plugin - you'd provide two of the same label (with different values). The idea here is that the artifact presents a state of truth about an image, but a specific preference needs to be customized at the selection (because maybe for my use case I want it differently).

### How would a request be made?

Given a constructed graph, a tool could use it against a request for some compatibility. In the simplest case, we require all attributes (`-r`):

```bash
compspec select <image-uri> --namespace supercontainers.org -r mpi.variant=openmpi -r hardware.gpu.enabled=yes
```

In the above example, note that we've also scoped the search to the "supercontainers.org" namespace, which immediately prunes off half the graph and makes search (and finding a match) easier. This also means we do not need to provide the full metadata attribute name (e.g., supercontainers.org.mpi.variant) in the request.  In the next example, we select required fields for three attributes:

```bash
compspec select <image-uri> \
    -r archspec.io.cpu.target=amd64 \
    -r supercontainers.org.mpi.variant=openmpi \
    -r supercontainers.org.hardware.gpu.enabled=yes
```

We might even, given an artifact has added custom metadata (basically another custom node in the graph) ask for that too.

```bash
compspec select <image-uri> \
    -r archspec.io.cpu.target=amd64 \
    -r supercontainers.org.mpi.variant=openmpi \
    -r mycustom.org.jellybean.flavor=pear
```

A more advanced tool that is able to be flexible about required attributes (e.g., try searching first for required and then retry without optional attributes given no matches) might take an optional flag `-o`:

```bash
compspec select <image-uri> \
    -r archspec.io.cpu.target=amd64 \
    -r supercontainers.org.mpi.variant=openmpi \
    -o mycustom.org.jellybean.flavor=buttered-popcorn
```

Indeed, nobody likes that flavor.

## Compatibility Plugins

While not a direct part of the proposal as it is unrelated to the artifact, it's worth discussing potential uses of the artifact toward compatibility plugins. We have [a prototype](https://github.com/compspec/compspec-go/blob/main/docs/usage) that demonstrates some of the ideas below.

!["Plugin Design](img/proposal-d-plugin-design.png "Plugin Design")

**Compatibility plugins** follow the interfaces that are dictated (and likely provided) by the OCI Compatibility working group. The exact design of this interface and development should be tackled by a follow-up working group once the path is decided here. This author (vsoch) offers to be a maintainer on these projects.

### Extraction and Checking Plugins

Three types of function interfaces are defined:

- An **extraction plugin** knows how to retrieve information about a system or environment.
- A **creation mapping** step knows how to create a compatibility artifact or JSON object for an image of interest (the artifact creation tool can use more than one at once). This assumes receiving information from extraction plugins and/or manual addition of metadata during CI builds and similar.
- A **checking plugin** has the methods or algorithms for comparing the extracted metadata (from an extraction plugin) to a compatibility artifact (from the creation plugin). This is where simple key/value checking or graph-based logic can happen. As long as the plugin provides this functionality via the common interfaces, the underlying logic is up to the designers of the plugin.

The details of these plugins and function interfaces should not be a prime concern for the document here, but by a follow up group. The above are provided for examples only. For the above, the compatibility interest groups likely define the checking plugins, and the OCI working groups (with expert groups) provide the underlying plugins for extraction or manifest generation.  A compatibility interest group could also provide an extraction plugin, if appropriate.

### Example Client

In the image above, an example client with subcommands is shown.

The artifact creation plugin takes one or more compatibility interest group creation plugins and can dump the results into one compatibility artifact. It also handles validation for correctness. A user requesting creation for an artifact will typically target one image, but can choose as many of the creation plugins as desired (e.g., "I want my artifact to be knowledgeable about architecture, GPU, etc).

The compatibility checking tool can be interacted with as a library or command line tool. It takes a request for one or more checking plugins to validate one or more container images for compatibility with a particular environment. Specifically:

- The onset of a check loads checking plugins of interest.
- Each checking plugin uses one or more extraction plugins to assess the system
- The system metadata "extraction" steps can be loaded once and shared between tools.

## Needed Community Projects

- Validation for the JGF (that all nodes are represented as edges in the graph)
- Visualization tools to map JGF to different formats for generating images or web interfaces

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
