# OmniBOR Specification

| Field   | Value |
|:--------|:------|
| Version | 0.1   |
| Status  | Draft |

## Foreword

This specification is subject to the Community Specification License 1.0,
available at <https://github.com/omnibor/spec>.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" are used as described in
[RFC 2119][rfc_2119].

Attention is drawn to the possibility that some of the elements of this
document may be the subject of patent rights. No party shall be held
responsible for identifying any or all such patent rights.

Any trade name used in this document is information given for the convenience
of users and does not constitute an endorsement.

This document was prepared by the OmniBOR Community.

Known patent licensing exclusions are available in the specification
repository's `NOTICES.md` file.

Any feedback or questions on this document should be directed to the
specification's repository, located at <https://github.com/omnibor/spec>.

THESE MATERIALS ARE PROVIDED "AS IS." The Contributors and Licensees expressly
disclaim any warranties (express, implied, or otherwise), including implied
warranties of merchantability, non-infringement, fitness for a particular
purpose, or title, related to the materials. The entire risk as to
implementing or otherwise using the materials is assumed by the implementer and
user. IN NO EVENT WILL THE CONTRIBUTORS OR LICENSEES BE LIABLE TO ANY OTHER
PARTY FOR LOST PROFITS OR ANY FORM OF INDIRECT, SPECIAL, INCIDENTAL, OR
CONSEQUENTIAL DAMAGES OF ANY CHARACTER FROM ANY CAUSES OF ACTION OF ANY KIND
WITH RESPECT TO THIS DELIVERABLE OR ITS GOVERNING AGREEMENT, WHETHER BASED ON
BREACH OF CONTRACT, TORT (INCLUDING NEGLIGENCE), OR OTHERWISE, AND WHETHER OR
NOT THE OTHER MEMBER HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

## Introduction

Software supply chains face many challenges: security and compliance chief
among them. Often, projects are hamstrung by the inability to easily and
reliably capture a complete, concise, verifiable accounting of exactly
__what__ inputs were built into software. Without this information,
identifying vulnerable software to patch or replace is difficult. While
Software Bills of Material (SBOMs) help identify third-party components,
they do not go far enough to precisely identify the exact inputs necessary
for vulnerability management.

The OmniBOR standard defines three concepts, which together enable the
consistent, reproducible, and embeddable encoding of the exact inputs used to
build a software artifact: Artifact Identifiers, Input Manifests, and Artifact
Dependency Graphs.

An Artifact Identifier is a content-based identifier of a single input (for
example, a single file) used to build a software artifact. Identifiers are
reproducible, meaning two individuals will always derive the same identifier
for the same input. With these identifiers, we can consistently and precisely
identify any software artifact or its input, for use in forensics, accounting,
and vulnerability management.

Next, an Input Manifest lists the Artifact Identifier of every input used to
produce an artifact. For example, if an executable is compiled by linking
together a collection of object files, the Artifact Identifier of every object
file would be listed in the Input Manifest for the executable. Input manifests
can be embedded directly into executable files, or can be provided in a
separate file alongside the artifact whose inputs they describe.

Finally, a collection of Input Manifests can be combined to produce an Artifact
Dependency Graph. The Artifact Dependency Graph is a complete description of
all inputs, direct or transitive, used to produce a software artifact.

Returning to the example of building an executable: the executable's Input
Manifest would list the Artifact Identifier of every object file, and each
object file would have its own Input Manifest listing the Artifact Identifier
of each of their source files. This set of Input Manifests can then be
resolved to produce an overall graph completely describing the inputs which
produced the executable.

With the Artifact Dependency Graph, consumers of this information could then
exactly identify when two artifacts were produced with exactly identical
inputs, and if inputs vary, could identify the exact inputs which vary and
observe how that affects the entirety of the graph. When coupled with
SBOM information about third-party dependencies, this can provide highly
specific and accurate identification of supply chain differences and their
causes.

This Artifact Dependency Graph may also be used to supplement vulnerability
information by precisely identifying affects files or resolving the impacts
of changes to those files across all users of those projects. By leveraging
transparent inclusion of Input Manifests into executable and other formats,
users would also gain the benefits of high precision supply chain information
without manually recording or updating those manifests as projects develop over
time.

##	Scope

Specifies procedures for constructing and conveying Artifact Dependency Graphs
(ADGs), and other related data structures for artifacts. Including but not
limited to:

- formats for artifact identifiers
- formats for specifying graph relationships between artifacts
- manner of embedding identifiers for ADGs, and other related
  data structures in artifacts of various types
- guidance on metadata which references ADGs, and other related data structures
- guidance for build tools for:
 - constructing ADGs, and other related data structures
 - conveying ADGs, and other related data structures
 - embedding identifiers for ADGs, and other related data structures ids in artifacts
 - manners of conveyance of ADGs, and other related data structures
 - descriptions of use cases for which ADGs, and other related data structures may be used

##	Normative References

- [GitOID URI][gitoid_uri]

## Terms and Definitions

For the purposes of this document, the following terms and definitions apply.

### Artifact 

An artifact is any object of interest that can be represented as arrays of
bytes (`[]byte`).

### Artifact Equivalency

Two artifacts are equivalent if and only if their binary representations are
equal. This can be expressed in pseudocode with the following expression:
`[]byte(artifact1) == []byte(artifact2)`

### Derived Artifacts

An artifact output from a build tool based on artifiacts input into the build
tool. Such an artifact is said to be 'derived' from the input artifacts to
the build tool.

### Leaf Artifacts

Artifacts which are not 'derived artifacts' are said to be 'leaf artifacts'.
Leaf artifacts are usually source code files constructed by hand by humans.

### Artifact Identifiers

It should be possible to identify each artifact with an artifact ID with
the following characteristics:

- __Canonical__: Independent parties, presented with equivalent artifacts,
  derive the same artifact identity.
- __Unique__: Non-equivalent artifacts have distinct identities.
- __Immutable__: An identified artifact can not be modified without also
  changing its identity.

### Artifact Dependency Graph (ADG)

The Artifact Dependency Graph (ADG) of an artifact is the recursive DAG
(Directed Acyclic Graph) of all the "input artifacts" that are transformed
by a build tool into that artifact. It includes the direct input artifacts,
and the recursive set of input artifacts to each input artifact, all the way
down the graph.

#### Artifact Dependency Graph (ADG) singularity

An artifact should have precisely one Artifact Dependency Graph (ADG). All
equivalent artifacts should have the same Artifact Dependency Graph (ADG).

## Build Tools

A build tool is something which reads one or more input artifacts and writes
one or more output artifacts.

## Specifications

### Artifact ID

Because two artifacts are equivalent if and only if their binary
representations are equal, a hash function may be applied to the binary
representation of an artifact to yield an identifier which satisfies the
canonical, unique, and immutable requirements of artifact identifiers.

### Artifact Identifier Types

The majority of source code artifacts are already stored in git and
indexed by their git object identifiers ("gitoids") as git objects of type
"blob".

For this reason, OmniBOR has chosen to use the "gitoid" of an Artifact as
its Artifact Identifier.

Git currently supports two varieties of gitoids. One is based on SHA1 and is
in common use. The other is based on SHA256 and has been very slow to garner
adoption. The [gitoid URI spec][gitoid_uri] uses different prefixes,
`gitoid:blob:sha1` or `gitoid:blob:sha256`, to distinguish which algorithm is
being used for computing the gitoid of a blob. This document adopts the gitoid
URI prefixes to distinguish Artifact Identifier Types. This approach is
anticipated to extend gracefully as git adopts new hash types in the future.

All subsequent references to mandatory identifier types in this document should
be interpreted to mean the list:

- `gitoid:blob:sha1`
- `gitoid:blob:sha256`

### OmniBOR Document

Am OmniBOR Document describes the immediate children of an artifact in the
Artifact Dependency Graph (ADG).

An OmniBOR Document utilizes precisely one Artifact Identifier Type.

### OmniBOR Identifier

An OmniBOR Document is identified by computing its identifier as an artifact
with the Artifact Identifier Type used for identifiers within the OmniBOR
Document itself.

#### OmniBOR Document Header

In order to distinguish the type of identifier used in the OmniBOR Document,
it begins with a single newline terminated header line:

```
${Artifact Identifier Type uri prefix}\n
```

For example:

```
gitoid:blob:sha1\n
```

or

```
gitoid:blob:sha256\n
```

All identifiers in a OmniBOR Document MUST be of the Artifact Identifier
Type declared in the header.

#### OmniBOR Document Child Records

The OmniBOR Document after the header consists of a list of newline terminated
child records

A child artifact which is itself a leaf artifacts would be represented by:

```
blob⎵${artifact id of child}\n
```

A child artifact which is itself a derived artifact would be represented by

```
blob⎵${artifact id of child}⎵bom⎵${OmniBOR identifier of child's OmniBOR Document}\n
```

The child artifact records must be written to the OmniBOR Document in lexical
order.

The artifact id and OmniBOR Document id must both be of the Artifact Identifier
Type declared in the OmniBOR Document header.

#### OmniBOR Document Character Encoding

All characters in a OmniBOR document are encoded in ASCII. Please note: all '\n'
must be encoded as '\n' characters, _not_ the line delimiter of the platform.

### OmniBOR Identifier Embedding

Each build tool should embed into the output artifact a new line terminated,
lexically ordered, list of OmniBOR identifiers for each mandatory Artifact
Identifier Type in a manner:

1. Appropriate to the type of artifact
2. Generally agreed upon for that artifact

### OmniBOR Document Construction by a Build Tool

A build tool creating an output artifact must compute a OmniBOR Document of
each mandatory artifact ID type.

For each input artifact the build tool must:

1. Compute the git object id of the input - `${artifact identifier}`
2. Examine the input for an embedded OmniBOR Document Identifier -
  `${omnibor document identifier}`

The build tool must persist a OmniBOR Document using the
`${artifact identifier}` and `${omnibor document identifier}` for each input.

## Bibliography

[rfc_2119]: https://tools.ietf.org/html/rfc2119
[gitoid_uri]: https://www.iana.org/assignments/uri-schemes/prov/gitoid
