# BiReUS - the Bidirectional Repository Update Service

**Important note:** BiReUS is not stable yet. File format compatibility may break during development.


BiReUS is a tool to create and apply binary patches for application data (versions forward and _optional_ backward) based on the bsdiff algorithm.

While the bsdiff algorthm itself can be applied on single files only, BiReUS can take care on a whole set of file repositories.


It aims to support the following use case:

*	You have a software (i.e. a computer game) where you have different kind of plugins (i.e. mods).
*	These plugins depend on lots of binary files.
  *	Binary files can also be zip-files which contain other text- or binary files.
*	These plugins get updated on a regular base following a versioning convention.
*	The software produces save-files (i.e. replays) which can be viewed later. However, to correctly open the save-file you need to have the exact the exact state of plugins in their original version.
*	The plugins are supposed to be stored on a server and retrieved on demand.
*	The software (client) handles the updates of the plugins using delta-files generated on the server.

BiReUS will only create and apply patches for you. It is intended to be run as a shell script or to be embedded inside other applications.

## Example
If you want to try it out, you can generate a demo repository using `python3 tests/create_test_server_data.py`.

You can now generate the patches using `python3 run-server.py -p tests/example-server`

The example contains all cases at least once (files/folders added, removed, unchanged, changed and zipped).


## Components

### Server
The server component scans all repositories in the given path and will generate patches for new versions to all existing versions.

Arguments:

* `-p <repository-path>` sets the repository path, _mandatory_
* `-c` or `--cleanup` removes all existing patches (but does not reset the .versions file)
* `-fo` or `--forward-only` skips generation of backward-patches

**Note:** For new repositories you need to create an info.json and a .versions file in the repository folder and add the first folder by hand.


### Client

The client component manages single repositories. It must be called from inside your desired working directory (either the top directory if you want to create a new one, or the repository itself if you want to checkout a version).

Arguments:
* `init <path> <url>` downloads the latest repository from an url to path
* `checkout` switches to the latest version
* `checkout <version>` switches to a specified version

**Note:** When checking out the latest version, the remote server is asked first. If it is not reachable, the latest local version will be checked out.


## .bireus - Specification
Every delta-zip contains a `.bireus`-file which describes the required actions to apply the patch.
`.bireus` is a JSON file with a unique header and a list of objects which may contain other objects recursively.

### Header
- **base_version:** the version this patch can be applied onto
- **target_version":** the version after applying the patch
- **repository:** the name of the repository
- **items:** list of delta-items

### Item
- **action:** contains the action how to patch this object
  - **add** for new files or folders that did not exist in the base version
  - **bsdiff** for files that have changed
  - **remove** for files or folders that were removed in the target version
  - **unchanged** for files that did not change
  - **zipdelta** for files that are actually zipfiles
    - zipdelta files have no checksums (due to different zip-parameter combinations). the checksums of the contents will be checked instead
- **name:** name of the file or directory
- **type:** file or directory
- **items:**
  - list of files or directories inside this directory
  - empty for type=file
- **base_crc:** _(only files)_ CRC32 of the original file
- **target_crc:** _(only files)_ CRC32 of the target file