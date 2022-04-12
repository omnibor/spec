 # GitBOM Specficiation
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

Specifies procedures for constructing and conveying artifact trees, directed acyclic graphs (DAGs), and other related 
data structures for artifacts.  Including but not limited to:

- formats for artifact identifiers
- formats for specifying graph relationships between artifacts
- manner of embedding identifiers for artifact trees, directed acyclic graphs (DAGs), and other related 
data structures in artifacts of various types
- guidance for build tools for:
  - constructing artifact artifact trees, DAGs, and other related data structures
  - conveying artifact trees, DAGs, and other related data structures
  - embedding identifiers for artifact trees, DAGs, and other related data structures ids in artifacts
  - manners of conveyance of artifact trees, DAGs, and other related data structures
  - descriptions of use cases for which artifact trees, directed acyclic graphs, and other related data structures may be used
  - guidance on metadata which references artifact trees, DAGs, and other related data structures

##	Normative references (mandatory)

There are no normative references in this document.

## Terms and definitions (mandatory)

For the purposes of this document, the following terms and definitions apply.

### Artifact 
An artifact is any object of interest that can be represented as arrays of bytes ([]byte).

### Artifact Equivalency

Two artifacts are equivalent if and only if `[]byte(artifact1) == []byte(artifact2)`

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

### Artifact Tree

The artifact tree of an artifact is the recursive DAG (Directed Acyclic Graph) of all the `input artifacts` that are transformed by a build tool] into that artifact.  It includes the direct input artifacts, and the recursive set of input artifacts to each input artifact, all the way down the tree.

#### Artifact tree singularity

An artifact should have precisely one artifact tree. All equivalent artifacts should have the same artifact tree.

## Build Tools

A build tool is something which reads one or more input artifacts and writes one or more output artifacts.

## Specifications
### Artifact ID

GitBOM uses the git object id (as a blob) for an artifact as its artifact ID.

### GitBOM Document

Each artifact has a GitBOM document that describes its immediate children consiting of a set of new line delimited records, one for each child, in lexical order.

A child artifact which is itself a leaf artifacts would be represented by

```
blob⎵${git object id of child}\n
```

A child artifact which is itself a derived artifact would be represented by
```
blob⎵${git object id of child}⎵bom⎵${git object id of child's GitBOM document}\n
```

### GitBOM identifier

For a given artifact, the GitBOM ID of that artifact is simply the git object id of it's GitBOM Document.

### GitBOM Identifier Embedding

Each build tool should embed the GitBOM ID of its output artifact(s) into the output artifact in a manner:

1.  Appropriate to the type of artifact
2.  Generally agreed upon for that artifact

### GitBOM Document Construction by a Build Tool

When a build tool reads an artifact, it should for each input file:
1. Compute the git object id of the input file - ${artifact identifier}
2. Examine the artifact for an embedded GitBOM identifier - ${gitbom idenfitier}
3. If the artifact contains a GitBOM identifer it construct a record:
   ```
   blob⎵${artifact identifier}⎵bom⎵${GitBOM identifier}\n
   ```
4. Otherwise construct a record:
   ```
   blob⎵${artifact identifier}\n
   ```

Concatenate all records together and apply lexical ordering by line to create the GitBOM document of the
output artifact.

### GitBOM Document persistence by a Build Tool

When persisting an output artifact to a file system, the build tool should create a subdirectory of the directory to which the output artifact is being written of the form:
```.bom/objects/``` and write out the GitBOM document to ```.bom/objects/${gitoid:0:2}/${gitoid:2:}``` where ```${gitoid}```
is the git object id of the GitBOM document.

### Build tool persistence of related metadata

When persisting an output artifact to a file system, if the build tool has additional metadata to persist that makes refernce to the artifact tree,
it should persist that metadata to a subdirectory of the directory to which the output artifact is being written of the form: ```.bom/metadata/${tool}/```.  Filenaming and subdirectory structure below that point is at the discretion of the build tool.
## Annex A


## Bibliography

