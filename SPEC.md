 # GitBOM Specification
## Version 1.0.0
## Status Pre-draft

 
This specification is subject to the Community Specification License 1.0, available at [https://github.com/git-bom/spec](https://github.com/git-bom/spec).

A guide on writing standards produced by the ISO/TMB and is available at https://www.iso.org/files/live/sites/isoorg/files/developing_standards/docs/en/how-to-write-standards.pdf.

A model manuscript of a draft International Standard (known as “The Rice Model”) is available at https://www.iso.org/iso/model_document-rice_model.pdf.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" as described in [RFC 2119](https://tools.ietf.org/html/rfc2119)

## Foreword

Attention is drawn to the possibility that some of the elements of this document may be the subject of patent rights. No party shall not be held responsible for identifying any or all such patent rights.

Any trade name used in this document is information given for the convenience of users and does not constitute an endorsement.

This document was prepared by the GitBOM Community.

Known patent licensing exclusions are available in the specification’s repository’s NOTICES.md file.

Any feedback or questions on this document should be directed to specifications repository, located at https://github.com/git-bom/spec.

THESE MATERIALS ARE PROVIDED “AS IS.” The Contributors and Licensees expressly disclaim any warranties (express, implied, or otherwise), including implied warranties of merchantability, non-infringement, fitness for a particular purpose, or title, related to the materials.  The entire risk as to implementing or otherwise using the materials is assumed by the implementer and user. IN NO EVENT WILL THE CONTRIBUTORS OR LICENSEES BE LIABLE TO ANY OTHER PARTY FOR LOST PROFITS OR ANY FORM OF INDIRECT, SPECIAL, INCIDENTAL, OR CONSEQUENTIAL DAMAGES OF ANY CHARACTER FROM ANY CAUSES OF ACTION OF ANY KIND WITH RESPECT TO THIS DELIVERABLE OR ITS GOVERNING AGREEMENT, WHETHER BASED ON BREACH OF CONTRACT, TORT (INCLUDING NEGLIGENCE), OR OTHERWISE, AND WHETHER OR NOT THE OTHER MEMBER HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

## Introduction
Type text.

##	Scope (mandatory)

Specifies procedures for constructing and conveying Artifact Dependency Graphs (ADGs), and other related
data structures for artifacts.  Including but not limited to:

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

##	Normative references (mandatory)

[gitoid uri](https://www.iana.org/assignments/uri-schemes/prov/gitoid)

## Terms and definitions (mandatory)

For the purposes of this document, the following terms and definitions apply.

### Artifact 
An artifact is any object of interest that can be represented as arrays of bytes ([]byte).

### Artifact Equivalency

Two artifacts are equivalent if and only if their binary representations are equal. This can be expressed in pseudocode with the following expression: []byte(artifact1) == []byte(artifact2)

### Derived Artifacts

An artifact output from a build tool based on artifiacts input into the build tool.  Such an artifact is said to be 
'derived' from the input artifacts to the build tool.

### Leaf Artifacts

Artifacts which are not 'derived artifacts' are said to be 'leaf artifacts'.
Leaf artifacts are usually source code files constructed by hand by humans.

### Artifact Identifiers

It should be possible to identify each artifact with an artifact ID with the following characteristics:

**Canonical**
: *Independent parties, presented with equivalent artifacts, derive the same artifact identity.*

**Unique**
: *Non-equivalent artifacts have distinct identities.*

**Immutable**
: *An identified artifact can not be modified without also changing its identity.*

### Artifact Dependency Graph (ADG)

The Artifact Dependency Graph (ADG) of an artifact is the recursive DAG (Directed Acyclic Graph) of all the `input artifacts` that are transformed by a build tool] into that artifact.  It includes the direct input artifacts, and the recursive set of input artifacts to each input artifact, all the way down the graph.

#### Artifact Dependency Graph (ADG) singularity

An artifact should have precisely one Artifact Dependency Graph (ADG). All equivalent artifacts should have the same Artifact Dependency Graph (ADG).

## Build Tools

A build tool is something which reads one or more input artifacts and writes one or more output artifacts.

## Specifications
### Artifact ID

Because two artifacts are equivalent if and only if their binary representations are equal, a hash function may be applied to the binary representation of an artifact to yield an identifier which satisfies the canonical, unique, and immutable requirements of artifact identifiers.

### Artifact Identifier Types
The vast majority of source code artifacts are already indexed by their git object identifiers (gitoids) as git objects of type blob.

For this reason GitBOM uses the gitoid for an artifact as its artifact ID.

Git currently supports two varieties of gitoids.  One is based on SHA1 and is in common use.  The other is based on SHA256 and has been very slow to garner adoption.  The [gitoid URI spec](https://www.iana.org/assignments/uri-schemes/prov/gitoid) uses different prefixes,  `gitoid:blob:sha1` or `gitoid:blob:sha256`, to distinguish which algorithm is being used for computing the gitoid of a blob.   This document adopts the gitoid uri prefixes to distinguish Artifact Identifier Types.  This approach is anticipated to extend gracefully as git adopts new hash types in the future.

All subsequent references to mandatory identifier types in this document should be interpreted to mean the list:

- gitoid:blob:sha1
- gitoid:blob:sha256

### GitBOM Document

A GitBOM Document describes the immediate children of an artifact in the Artifact Dependency Graph (ADG).

A GitBOM Document utilizes precisely one Artifact Identifier Type.

### GitBOM Identifier

A GitBOM Document is identified by computing its identifier as an artifact with the Artifact Identifier Type used for identifiers within the GitBOM Document itself.

#### GitBOM Document Header

In order to distinguish the type of identifier used in the GitBOM Document, it begins with a single newline terminated header line:

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

All identifiers in a GitBOM Document MUST be of the Artifact Identifier Type declared in the header.

#### GitBOM Document Child Records

The GitBOM Document after the header consists of a list of newline terminated child records

A child artifact which is itself a leaf artifacts would be represented by

```
blob⎵${artifact id of child}\n
```

A child artifact which is itself a derived artifact would be represented by
```
blob⎵${artifact id of child}⎵bom⎵${GitBOM identifier of child's GitBOM Document}\n
```

The child artifact records must be written to the GitBOM Document in lexical order.

The artifact id and GitBOM Document id must both be of the Artifact Identifier Type declared in the GitBOM Document header.

### GitBOM Identifier Embedding

Each build tool should embed into the output artifact a new line terminated, lexically ordered, list of GitBOM identifiers for each mandatory Artifact Identifier Type in a manner:

1. Appropriate to the type of artifact
2. Generally agreed upon for that artifact

### GitBOM Document Construction by a Build Tool

A build tool creating an output artifact must compute a GitBOM Document of each mandatory artifact id type.

For each input artifact the build tool must:

1. Compute the git object id of the input - ${artifact identifier}
2. Examine the input for an embedded GitBOM Document Identifier - ${gitbom document identifier}

The build tool must persist a GitBOM Document using the ${artifact identifier} and
${gitbom document identifier} for each input.

### GitBOM Document persistence by a Build Tool

Each build tool should persist all GitBOM Documents it generates.

Generically, GitBOM persistence can be thought of as having an abstract API:
```
WriteGitBOM(GitBOM id,GitBOM contents)
ReadGitBOM(GitBOM id) returns GitBOM Contents
```

WriteGitBOM should:

- Check that GitBOM id matches GitBOM contents and return an error if it does not.
- Check that the GitBOM Document follows the GitBOM Document format and return an error if it does not.
## Annex A


## Bibliography

