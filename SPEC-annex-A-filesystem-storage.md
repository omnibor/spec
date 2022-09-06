## Annex A

Annex A documents known methods of persisting GitBOM Documents to various stores.
### GitBOM Document persistence by a Build Tool to its local filesystem

If a build tool persists GitBOM information to its local filesystem, the build tool should write out the GitBOM Document to ```${GITBOM_DIR}/objects/${Artifact Identifier Type uri prefix with ':' replaced by '/'}/${GitBOMID:0:2}/${GitBOMID:2:}``` where ```${artifact id}``` is the GitBOM id in lowercase hexidecimal.

Example:

```
.adg/objects/gitoid/blob/sha1/0e/8efd4cdf0d5bafcfcae658c2662a73b199b301
```

#### Build tool persistence of related metadata

If the build tool has additional metadata to persist that makes referece to the Artifact Dependency Graph (ADG),
it should persist that metadata to a subdirectory of the directory to which the output artifact is being written of the form: ```${GITBOM_DIR}/metadata/${tool}/```.  Filenaming and subdirectory structure below that point is at the discretion of the build tool.

#### Build tool selection of GITBOM_DIR
GITBOM_DIR should be set in order of precedence by:
1.  A non-empty env variable named GITBOM_DIR
2.  A build tool specific flag
3.  A subdirectory ```.adg/``` of the directory to which it is writing out the artifact

- The presence of an empty GITBOM_DIR env var should be taken as a signal to skip gitbom generation
- The lack of a GITBOM_DIR env var should not be taken as a signal to _not_ generate a gitbom.

