Scheme name: gitoid

Status: provisional

Applications/protocols that use this scheme name:
     See [Section 3.5](https://www.rfc-editor.org/rfc/rfc7595.html#section-3.5).
Contact:
     Registering party: Ed Warnicke <eaw@cisco.com>

Scheme Creator: GitBOM

Change controller:
    Either the registering party or someone who is verified to represent
    the scheme creator.  See previous answer.

References:
https://git-scm.com/book/en/v2/Git-Internals-Git-Objects

Scheme syntax: gitoid":"<git object type>":"<hash algorithm>":"<hash value>

Current valid values for <git object type> are 'blob','tree','commit', and 'tag'.
Current valid values for <hash algorithm> are 'sha1' and 'sha256'
<hash value> should be expressed as a hexidecimal string in lower case.

Example: gitoid:blob:sha1:261eeb9e9f8b2b4b0d119366dda99c6fd7d35c64

Scheme semantics:

gitoid stands for Git Object ID.  Git is an object store.  It currently has four types of objects.  A git object is formed by prepending an object header to the contents of the object.  A git object header consists of:

<git object type>" "<size of contents as decimal string>"\0"

The git object id is computed by applying a hash to the git object.  Currently the most common hash is sha1, but there is some effort to transition to sha256 over time.

A gitoid URI identifies a git object independent of any particular git repository.  Given a byte array and a gitoid, it should be possible to construct the analogous git object header from the gitoid, as gitoid contains the git object type.  By prepending the constructed git object header to the byte array and performing the hash indicated in the gitoid, it should be possible to determine whether or not the gitoid matches the byte array.

Encoding considerations:
gitoid's should be encoded in utf-8, which for the range of valid characters is also ascii.

Interoperability considerations:
Unknown, use with care.

Security considerations:
Unknown, use with care.