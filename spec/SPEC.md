# OmniBOR Specification

| Field   | Value |
|:--------|:------|
| Version | 0.2   |
| Status  | Draft |

## 1. Foreword

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

## 2. Introduction

The OmniBOR standard defines two concepts, which together enable the
consistent, reproducible, and embeddable identification of software
artifacts and the encoding of the exact inputs used to build software
artifacts: Artifact Identifiers and Input Manifests.

An Artifact Identifier is a content-based identifier of a single input (for
example, a single file) used to build a software artifact. Identifiers are
reproducible, meaning two individuals will always derive the same identifier
for the same input. With these identifiers, we can consistently and precisely
identify any software artifact or its input, for use in forensics, accounting,
and vulnerability management.

Next, an Input Manifest lists the Artifact Identifier of every input used to
produce an artifact. For example, if an executable is compiled by linking
together a collection of object files, the Artifact Identifier of every object
file would be listed in the Input Manifest for the executable. Input Manifests
can be identified by their own Artifact Identifier. The Artifact ID for the
manifest can be embedded directly into executable files, or can be provided
in a separate file alongside the artifact whose inputs they describe.

The central purpose in the design of Artifact Identifiers and Input Manifests
is to enable creation of Artifact Dependency Graphs. An Artifact Dependency
Graph is a fine-grained description of all inputs, direct or transitive, used
to produce a software artifact. This graph can be constructed from collections
of Input Manifests for all dependencies used in an artifact's construction.

Returning to the example of building an executable: the executable's Input
Manifest would list the Artifact Identifier of every object file, and each
object file would have its own Input Manifest listing the Artifact Identifier
of each of their source files. This set of Input Manifests can then be
resolved to produce an overall graph completely describing the inputs which
produced the executable.

With the Artifact Dependency Graph, consumers of this information could then
exactly identify when two artifacts were produced with exactly identical
inputs, and if inputs vary, could identify the exact inputs which vary and
observe how that affects the entirety of the graph. Changes in inputs anywhere
in the graph would result in new Artifact Identifiers for the changed input
and all inputs derived from it, enabling easy detection of changes in
the graph.

This Artifact Dependency Graph may also be used to supplement vulnerability
information by precisely identifying affected files or resolving the impacts
of changes to those files across all users of those projects. By leveraging
transparent inclusion of Input Manifests into executable and other formats,
users would also gain the benefits of high precision dependency information
without manually recording or updating those manifests as projects develop over
time.

## 3. Scope

This document specifies:

- The in-memory format of Artifact Identifiers
- The textual representation of Artifact Identifiers
- The process for constructing Artifact Identifiers
- The textual representation of Input Manifests
- How Input Manifests should be stored in a file system
- How Input Manifests should be embedded into the artifacts whose build
  inputs they are describing
- How Input Manifests should be constructed by build tools.

##	4. Normative References

- [GitOID URI][gitoid_uri]

## 5. Terms and Definitions

For the purposes of this document, the following terms and definitions apply.

### 5.1. Artifact

An artifact is any object of interest that can be represented as an array of
bytes.

### 5.2. Artifact Equivalency

Two artifacts are equivalent if and only if their binary representations are
equal.

### 5.3. Build Tools

A build tool is anything which reads one or more input artifacts and writes
one or more output artifacts. Examples of build tools include:

- __Compilers:__
  - `llvm-clang`
  - `gcc`
  - `javac`
  - `rustc`
  - `go`
- __Linkers:__
  - `llvm-lld`
  - `binutils-ld`
- __Runtimes__
  - Java JVM
  - Node.js
  - Python interpreter
- __Code Generators__

## 6. Specifications

### 6.1. Artifact Identifier

It MUST be possible to identify each artifact with an Artifact Identifier with
the following characteristics:

- __Reproducible__: Independent parties, presented with equivalent artifacts,
  derive the same Artifact Identifier.
- __Unique__: Non-equivalent artifacts have distinct Identifiers.
- __Immutable__: An identified artifact cannot be modified without also
  changing its Identifier.

Artifact Identifier can be shortened to Artifact ID.

Because two artifacts are equivalent if and only if their binary
representations are equal, a hash function may be applied to the binary
representation of an artifact to yield an identifier which satisfies the
reproducible, unique, and immutable requirements of Artifact Identifiers.

### 6.2. Artifact Identifier Types

The majority of source code artifacts are already stored in Git and
indexed by their Git Object Identifiers ("GitOIDs") as Git objects of type
"blob".

For this reason, OmniBOR has chosen to use the "GitOid" of an Artifact as
its Artifact Identifier.

Git currently supports two varieties of GitOIDs. One is based on SHA-1 and is
in common use. The other is based on SHA-256 and has been slow to garner
adoption.

Git's use of SHA-1 is additionally complicated by the fact that Git actually
uses a variant of SHA-1 in newer versions, called SHA-1CD (where "CD" stands
for "collision detection") which tries to detect attempts to engineer
purposeful hash collisions in SHA-1 and subverts them by modifying the
operation of the hash algorithm in those cases. Git calls this "SHA-1" and does
not frequently distinguish use of the two similar but not equivalent hash
algorithms.

The [GitOID URI spec][gitoid_uri] uses different prefixes,
`gitoid:blob:sha1` or `gitoid:blob:sha256`, to distinguish which algorithm is
being used for computing the GitOID of an artifact, subject to the knowledge
that `gitoid:blob:sha1` may describe either SHA-1 or SHA-1CD depending on the
version of Git being used.

This document adopts the GitOID URI prefixes to distinguish Artifact Identifier
Types. This approach is anticipated to extend gracefully as Git adopts new hash
types in the future.

Given the challenges with SHA-1 including:

- Its weakness as a hash algorithm today, with some attacks already being
  known publicly which permit collisions in some contexts,
- The fact that Git itself is expending effort to transition away from the use
  of SHA-1 and toward SHA-256,
- A concern that SHA-1 could in the next five years be subject to orders to
  transition away by some world governments,

We have decided to only permit the use of SHA-256 as a hash algorithm for
Artifact Identifiers for OmniBOR.

We reserve the right to extend this list to support additional hash algorithms
in the future, for example if SHA-256 is determined to be broken by future
computing capabilities.

All subsequent references to Artifact Identifier types in this document should
be interpreted to mean the list:

- `gitoid:blob:sha256`

### 6.3. Input Manifest

An Input Manifest for an artifact enumerates the inputs to the build tool that
produced the artifact.

A given Input Manifest utilizes precisely one Artifact Identifier Type.

#### 6.3.1. Input Manifest Header

In order to distinguish the type of identifier used in the Input Manifest,
it begins with a single newline-terminated header line:

```
${Artifact Identifier Type URI prefix}\n
```

For example:

```
gitoid:blob:sha256\n
```

All identifiers in a Input Manifest MUST be of the Artifact Identifier
Type declared in the header.

#### 6.3.2. Input Manifest Records

The Input Manifest after the header consists of a list of newline terminated
input records.

Each input record consists of:

- The Artifact ID of the input artifact
- Optionally, the Artifact ID of the Input Manifest for the input artifact

An Input Manifest record for an artifact for which no Input Manifest
is known is represented as:

```
${artifact identifier of the input artifact}\n
```

An Input Manifest record for an artifact:

```
${artifact identifier of the input artifact}⎵manifest⎵${artifact identifier of the input manifest for the input artifact}\n
```

`⎵` above refers to the ASCII space character (0x20).

Artifact Identifiers in Input Manifests should be represented as a string in
lowercase hexadecimal. For example:

```
514516097a2f95c893f2a9685bcecfb85b7598e6
```

The input artifact records MUST be written to the Input Manifest in lexical
order. This is defined by sorting primarily by the input type, and secondarily
by the Artifact ID of the input artifact.

The Artifact Identifier for the input artifact and for the input artifact's
Input Manifest MUST both be of the Artifact Identifier Type declared in the
Input Manifest header.

#### 6.3.3. Input Manifest Character Encoding

All characters in an Input Manifest are encoded in ASCII. Please note: all '\n'
MUST be encoded as '\n' characters, _not_ the line delimiter of the platform.
This is necessary because the Input Manifest will be hashed to produce its
Artifact Identifier, and these Artifact Identifiers MUST be consistent
regardless of the platform on which the Input Manifest generation is performed.

#### 6.3.4. Input Manifest Embedding

Each build tool SHOULD embed into the output artifact a deterministically
ordered list of Artifact IDs for the Input Manifest for each mandatory Artifact
Identifier Type in a manner:

1. Appropriate to the type of artifact
2. Generally agreed upon for that artifact

If embedding is not possible — for example, if the format of the output
artifact does not permit a method to embed additional information without
breaking the functionality of that artifact — then embedding SHOULD be
skipped.

#### 6.3.5. Input Manifest Construction

A build tool creating an output artifact MUST compute an Input Manifest of
each mandatory Artifact Identifier Type.

For each input artifact the build tool MUST:

1. Compute the artifact identifier of the input: `${artifact identifier}`
2. Examine the input for an embedded Artifact ID for an Input Manifest:
   `${input manifest artifact id}`

The build tool MUST persist an Input Manifest using the
`${artifact identifier}` and `${input manifest artifact id}` for each input.

#### 6.3.6. Input Manifest Example

```
gitoid:blob:sha256
09c825ac02df9150e4f93d12ba1da5d1ff5846c3e62503c814aa3a300c535772
230f3515d1306690815bd9c3da0d15d8b6fcf43894d17100eb44b6d329a92f61
2f4a51b16b76bbc87c4c27af8ae062b1b50b280f1ab78e3eec155334588dc88e manifest 4f3a822f776412c049dda53c3277bf2225b51b805ce8a99222af23a7d9f55636
c71d239df91726fc519c6eb72d318ec65820627232b2f796219e87dcf35d0ab4
f47ffb3518f236eea6525fd29f057ddd5cda1bb803ccc662e6bc5925afd1e4af
```

## 7. Storing Input Manifests

This section documents known methods of persisting Input Manifests to
various stores.

If a build tool persists an Input Manifest to its local filesystem,
the build tool should write out the Input Manifest to
`${OMNIBOR_DIR}/manifests/${Artifact Identifier Type URI prefix with ':' replaced by '_'}/${Input Manifest Artifact ID:0:2}/${Input Manifest Artifact ID:2:}`
where `${Input Manifest Artifact ID}` is the Artifact ID of the Input
Manifest in lowercase hexadecimal with leading zeros NOT suppressed.

Example:

If `OMNIBOR_DIR=.omnibor` then the Input Manifest with Artifact ID
`gitoid:blob:sha256:09c825ac02df9150e4f93d12ba1da5d1ff5846c3e62503c814aa3a300c535772`
would be stored in:

```
.omnibor/manifests/gitoid_blob_sha256/09/c825ac02df9150e4f93d12ba1da5d1ff5846c3e62503c814aa3a300c535772
```

### 7.1. Use of a Target Index

Some desirable operations related to Input Manifests require associating an
Input Manifest with the artifact it is describing. Ideally, the artifact itself
has embedded in it the Artifact ID of its Input Manifest. However, in cases
where this is not true, or as a performance optimization even when it _is_ true,
it is desirable to maintain a "Target Index" which associated the Artifact ID
of an artifact with the Artifact ID of its Input Manifest.

This Target Index MUST be stored at `${OMNIBOR_DIR}/targets`, and MUST take
the form of a text file where each line (separated only by the `\n` character)
MUST be formatted as follows:

```
${Artifact ID of the target artifact}⎵${Artifact ID of the Input Manifest}
```

`⎵` above refers to the ASCII space character (0x20).

This index file may then be used to improve the performance and reliability of
operations related to Input Manifests.

### 7.2. Selection of Storage Location

The storage location for Input Manifests MAY be set by the following methods,
listed in order of increasing precedence:

1. A non-empty env variable named `OMNIBOR_DIR`
2. A build tool specific flag

The absence of specification of a storage location via either the build tool
specific flag or `OMNIBOR_DIR` variable MAY be taken as a signal to skip
generation of Input Manifests by build tools.

## 8. Embedding Artifact IDs in ELF Files

This section contains a method of embedding Artifact IDs for Input Manifests
into ELF files.

If an ELF artifact contains an embedded Artifact ID for an Input Manifest,
then implementations MUST conform to the format specified in this document.

Artifact IDs for Input Manifests MUST be persisted by build tools when they
build an artifact and produce an Input Manifest for that artifact.

When persisting Artifact IDs for Input Manifests into an ELF object or an ELF
executable, the build tool MUST create a [section][elf_section] named
`.note.omnibor` and place the Artifact IDs in the descriptor field of the note
entry.

This section MUST be of type `SHT_NOTE` and MUST have the attribute
`SHF_ALLOC`. Multiple Note entries MAY be created, one for each Artifact
Identifier Type used.

Each note entry MUST contain the following fields in the same order as given
below:

1. `namesz` (4 bytes): This field MUST be set to a value of `8`, the length of
   the 'owner' field `OMNIBOR\0` in bytes.
2. `descz` (4 bytes): This field MUST contain the length of the Artifact ID for
   the Input Manifest in bytes, including a byte for the null terminator.
3. `type` (4 bytes): This field MUST contain the value associated with one of
   the reserved Artifact Identifier types. The values for the reserved types
   are in the range of `0x00000000` to `0x7fffffff`. Permissible types with
   reserved values are:
   - `NT_GITOID_BLOB_SHA256` is `0x1`
4. `owner` (8 bytes): This field MUST contain the string `OMNIBOR\0`, padded
   to 8 bytes.
5. `descriptor`: This field MUST contain the Artifact IDs for the Input
   Manifests as raw bytes. The length of this field is the same as the value
   in the `descz` field.

When recording multiple Artifact IDs for Input Manifests in the note section,

1. There MUST be only one note entry for each Artifact Identifier Type.
2. The note entries MUST be in ascending order of Artifact Identifier Type.

Build tools MUST generate all Artifact Identifier Types, currently only SHA-256.

## 9. Embedding Artifact IDs in Text Files

This section contains a method of embedding an Artifact ID for an Input
Manifest into source code files.

Most source code files are hand coded by humans. Some however are generated
from other input(s) by a build tool.

A build tool outputing a source code file may embed the Artifact ID for the
Input Manifest for the output source code file into the output source code
file by adding a comment line containing a string of the form:

```
OmniBOR-Input-Manifests: [ ${comma separated list of Artifact ID URIs for Input Manifests} ]
```

For a file with C commenting semantics (like C, C++, Java, Go, etc) a concrete
example might be:

```
// OmniBOR-Input-Manifests: [ gitoid:blob:sha256:09c825ac02df9150e4f93d12ba1da5d1ff5846c3e62503c814aa3a300c535772 ]
```

For a file with shell commenting semantics (like a shell script, Python, etc) a
concrete example might be:

```
# OmniBOR-Input-Manifests: [ gitoid:blob:sha256:09c825ac02df9150e4f93d12ba1da5d1ff5846c3e62503c814aa3a300c535772 ]
```

When interpretting an OmniBOR-Input-Manifest-ID comment line a reader should
ignore any leading and trailing spaces around `[` or `]` or `,`.

### 9.1. Placement of Embedded Artifact IDs

The `OmniBOR-Input-Manifest` comment line should be placed as the last line in
the source code file. The `OmniBOR-Input-Manifest` comment line should be
preceded by a blank line to ensure it is not interpretted as part of another
comment block.

A tool reading the source code file should interpret the last
`OmniBOR-Input-Manifest` comment line it encounters in the file as being the
Artifact ID of the Input Manifest, and ignore previous comment lines in the
file which may contain additional Artifact IDs.

Example:

If the input source code file begins with:

```go
// Code generated by stringer DO NOT EDIT.

import (
    "fmt"
)
...
```

The output source code file should look like:
```go
// Code generated by stringer DO NOT EDIT.

import (
    "fmt"
)
...

// OmniBOR-Input-Manifest: [ gitoid:blob:sha256:09c825ac02df9150e4f93d12ba1da5d1ff5846c3e62503c814aa3a300c535772 ]
```

If the input source code file begins with:

```c
/*
 * Copyright 2023 Yoyodyne Inc
 * SPDX-License-Identifier:
 */

#include <stdio.h>
int main() {
   // printf() displays the string inside quotation
   printf("Hello, World!");
   return 0;
}
```

The output source code file should look like:

```c
/*
 * Copyright 2023 Yoyodyne Inc
 * SPDX-License-Identifier:
 */

#include <stdio.h>
int main() {
   // printf() displays the string inside quotation
   printf("Hello, World!");
   return 0;
}

// OmniBOR-Input-Manifest: [ gitoid:blob:sha256:09c825ac02df9150e4f93d12ba1da5d1ff5846c3e62503c814aa3a300c535772 ]
```

### 9.2. Modifying Existing Text Files

Many source code generation tools, like `patch`, specifically mutate an
existing input source code file which may contain an existing
`OmniBOR-Input-Manifest` comment. In such circumstances the tool SHOULD
either

1. Replace an existing `OmniBOR-Input-Manifest` comment if found
2. Insert the `OmniBOR-Input-Manifest` normally, which will cause it to
   be placed _after_ the existing `OmniBOR-Input-Manifest` comment line.

[elf_section]: https://refspecs.linuxfoundation.org/LSB_3.0.0/LSB-PDA/LSB-PDA.junk/sections.html
[rfc_2119]: https://tools.ietf.org/html/rfc2119
[gitoid_uri]: https://www.iana.org/assignments/uri-schemes/prov/gitoid
