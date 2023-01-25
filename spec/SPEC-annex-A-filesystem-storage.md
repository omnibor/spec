## Annex A

Annex A documents known methods of persisting OmniBOR Documents to various stores.
### OmniBOR Document persistence by a Build Tool to its local filesystem

If a build tool persists OmniBOR information to its local filesystem, the build tool should write out the OmniBOR Document to ```${OMNIBOR_DIR}/objects/${Artifact Identifier Type uri prefix with ':' replaced by '_'}/${OmniBORID:0:2}/${OmniBORID:2:}``` where ```${OmniBORID}``` is the OmniBOR id in lowercase hexidecimal without leading zeros NOT suppressed.

Example:

```
.adg/objects/gitoid_blob_sha1/0e/8efd4cdf0d5bafcfcae658c2662a73b199b301
```

#### Build tool persistence of related metadata

A build tool may persist additional metadata to that makes reference to the Artifact Dependency Graph (ADG).
It should persist such metadata to a subdirectory of the directory to which the output artifact is being written of the form: ```${OMNIBOR_DIR}/metadata/${context}/```.  

For metadata specific to a particular build tool ```${context}``` should be a name uniquely associated with the build tool.  Build tools should report their selection of ```${context}``` subdirectory name to the OmniBOR spec for inclusion in a list to preclude ```${context}``` collision.

Metadata persisted by multiple build tools in the same way should be documented in a specification for that metadata.  Such specs must include the ```${context}``` for that metadata.  Such specs should be reported to the OmniBOR spec for inclusion in a list to preclude ```${context}``` collision.

Subdirectory structure, filenaming, and file schema below that point are at the discretion of the build tool for build tool specific metadata or the metadata spec for common metadata.

#### Build tool selection of OMNIBOR_DIR

OMNIBOR_DIR should be set in order of precedence by:
1.  A non-empty env variable named OMNIBOR_DIR
2.  A build tool specific flag

- The absence of specification of an OMNIBOR_DIR should be taken as a signal to skip OmniBOR generation.
