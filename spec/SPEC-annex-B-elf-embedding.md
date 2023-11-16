# Annex B

Annex B contains a method of embedding Input Manifest Identifiers into ELF
files.

## Input Manifest Identifiers

Input Manifest Identifiers are Artifact Identifiers (Git Object Identifiers
\[GitOIDs\]) for Artifact Input Manifests. They identify an Artifact Input
Manifest and MAY be embedded into an artifact to relate the artifact to its
Artifact Input Manifests.

If an ELF artifact contains an embedded Input Manifest Identifier, then
implementations MUST conform to the format specified in this document.

Note that multiple Input Manifests MUST be produced for a single artifact,
reflecting the use of different hash functions to produce the Artifact
Identifiers.

### Input Manifest Identifier persistence in ELF Objects/Executables

Input Manifest Identifiers MUST be persisted by build tools when they build
an artifact and produce an Artifact Input Manifest for that artifact.

When persisting Input Manifest Identifiers into an ELF object or an ELF
executable, the build tool MUST create a [section][elf_section]
`.note.omnibor` and place the Input Manifest Identifiers in the descriptor
field of the note entry. This section MUST be of type `SHT_NOTE` and MUST have
the attribute `SHF_ALLOC`. Multiple Note entries MUST be created, one for each
Artifact Identifier type when multiple Artifact Identifier types are involved.
Each note entry MUST contain the following fields in the same order as given
below:

1. `namesz` (4 bytes): This field MUST be set to a value of `8`, the length of
   the 'owner' field `OMNIBOR\0` in bytes.
2. `descz` (4 bytes): This field MUST contain the length of the Input Manifest
   Identifier in bytes, including a byte for the null terminator.
3. `type` (4 bytes): This field MUST contain the value associated with one of
   the reserved Artifact Identifier types. The values for the reserved types
   are in the range of `0x00000000` to `0x7fffffff`. Permissible types with
   reserved values are:

   ```
	NT_GITOID_BLOB_SHA1 = 0x1,
	NT_GITOID_BLOB_SHA256 = 0x2,
   ```

4. `owner` (8 bytes): This field MUST contain the string `OMNIBOR\0`, padded to
   8 bytes.
5. `descriptor`: This field MUST contain the Input Manifest Identifiers as raw
   bytes.The length of this field is the same as the value in the `descz` field.

When recording multiple Input Manifest Identifiers in the note section,

1. There MUST be only one note entry for each Input Manifest Identifier type.
2. The note entries MUST be in ascending order of Input Manifest Identifier
   type.

Conforming build tools MUST generate all Input Manifest Identifier types,
currently SHA1 and SHA256 Artifact Identifiers.

[elf_section]: https://refspecs.linuxfoundation.org/LSB_3.0.0/LSB-PDA/LSB-PDA.junk/sections.html
