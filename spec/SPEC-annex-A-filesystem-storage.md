## Annex A

Annex A documents known methods of persisting OmniBOR Documents to various stores.

### Input Manifest persistence by a Build Tool to its local filesystem

If a build tool persists an Input Manifest to its local filesystem, the build tool should write out the Input Manifest to ```${OMNIBOR_DIR}/objects/${Artifact Identifier Type uri prefix with ':' replaced by '_'}/${Input Manifest Identifier:0:2}/${Input Manifest Identifier:2:}``` where ```${Input Manifest Identifier}``` is Input Manifest Identifier in lowercase hexadecimal with leading zeros NOT suppressed.

Example:

If ```OMNIBOR_DIR=.omnibor``` then the Input Manifest with ```gitoid:blob:sha1`` Input Manifest Identifier
```0e8efd4cdf0d5bafcfcae658c2662a73b199b301``` would be stored in:

```
.omnibor/objects/gitoid_blob_sha1/0e/8efd4cdf0d5bafcfcae658c2662a73b199b301
```

#### Build tool persistence of related metadata

A build tool may persist additional metadata to that makes reference to the Artifact Dependency Graph (ADG).
It should persist such metadata to a subdirectory of the directory to which the output artifact is being written of the form: ```${OMNIBOR_DIR}/metadata/${context}/```.  

For metadata specific to a particular build tool ```${context}``` should be a name uniquely associated with the build tool.  For example: 

- ```${OMNIBOR_DIR}/metadata/llvm```
- ```${OMNIBOR_DIR}/metadata/clang```
- ```${OMNIBOR_DIR}/metadata/go```
- ```${OMNIBOR_DIR}/metadata/rustc```
- ```${OMNIBOR_DIR}/metadata/gcc```

Build tools should report their selection of ```${context}``` subdirectory name to the OmniBOR spec for inclusion in a list to preclude ```${context}``` collision.

Metadata persisted by multiple build tools in the same way should be documented in a specification for that metadata.  Such specs must include the ```${context}``` for that metadata.  Such specs should be reported to the OmniBOR spec for inclusion in a list to preclude ```${context}``` collision.  For example, if a group of build tools decide to store metadata about file locations in a common format, they might choose to define a ```${context}``` ```filelocation``` in which case the metadata would be stored in ```${OMNIBOR_DIR}/metadata/filelocation```

Subdirectory structure, filenaming, and file schema below ```${OMNIBOR_DIR}/metadata/${context}/``` are at the discretion of the build tool for build tool specific metadata or the metadata spec for common metadata.

#### Build tool selection of OMNIBOR_DIR

OMNIBOR_DIR may be set by the following methods, listed in order of precedence:
1.  A non-empty env variable named OMNIBOR_DIR
1. A build tool specific flag
2. A non-empty env variable named OMNIBOR_DIR

The absence of specification of a location to write omnibor data via either the OMNIBOR_DIR or the build tool specific flag  may be taken as a signal to skip OmniBOR generation.
