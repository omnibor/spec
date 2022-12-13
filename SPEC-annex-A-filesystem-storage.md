## Annex A

Annex A documents known methods of persisting GitBOM Documents to various stores.
### GitBOM Document persistence by a Build Tool to its local filesystem

If a build tool persists GitBOM information to its local filesystem, the build tool should write out the GitBOM Document to ```${GITBOM_DIR}/objects/${Artifact Identifier Type uri prefix with ':' replaced by '_'}/${GitBOMID:0:2}/${GitBOMID:2:}``` where ```${GitBOMID}``` is the GitBOM id in lowercase hexidecimal without leading zeros NOT suppressed.

Example:

```
.adg/objects/gitoid_blob_sha1/0e/8efd4cdf0d5bafcfcae658c2662a73b199b301
```

#### Build tool persistence of artifact to GitBOM Document mapping

A build tool should choose to persist a mapping between artifacts and their corresponding GitBOM documents.  If it chooses
to do so it should for each artifact persist a symlink:

```${GITBOM_DIR}/a2g/${Artifact Identifier Type uri prefix with ':' replaced by '_'}/${artifact id}:0:2}/${artifact id:2:} -> ${relative symlink to GitBOM document for artifact}```

`a2g` is short for `artifact to graph`.

Example:

```
.adg/a2g/gitoid_blob_sha1/0e/8efd4cdf0d5bafcfcae658c2662a73b199b301 -> ../../../../../objects/gitoid/blob/sha1/1d/6e79da5e380e5d3e5adcf899c4d65c0e80bfb3
```

#### Build tool persistence of related metadata

A build tool may persist additional metadata to that makes reference to the Artifact Dependency Graph (ADG).
It should persist such metadata to a subdirectory of the directory to which the output artifact is being written of the form: ```${GITBOM_DIR}/metadata/${context}/```.  

For metadata specific to a particular build tool ```${context}``` should be a name uniquely associated with the build tool.  Build tools should report their selection of ```${context}``` subdirectory name to the GitBOM spec for inclusion in a list to preclude ```${context}``` collision.

Metadata persisted by multiple build tools in the same way should be documented in a specification for that metadata.  Such specs must include the ```${context}``` for that metadata.  Such specs should be reported to the GitBOM spec for inclusion in a list to preclude ```${context}``` collision.

Subdirectory structure, filenaming, and file schema below that point are at the discretion of the build tool for build tool specific metadata or the metadata spec for common metadata.

#### Build tool selection of GITBOM_DIR

GITBOM_DIR should be set in order of precedence by:
1.  A non-empty env variable named GITBOM_DIR
2.  A build tool specific flag
3.  A subdirectory ```.adg/``` of the directory to which it is writing out the artifact

- The presence of an empty GITBOM_DIR env var should be taken as a signal to skip gitbom generation
- The lack of a GITBOM_DIR env var should not be taken as a signal to _not_ generate a gitbom.

