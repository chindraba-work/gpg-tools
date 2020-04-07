# gpg-tools

## Contents

- [Description](#description)
  * [Usage of `certsplitter`](#usage-of-certsplitter)
  * [Usage of `keyimporter`](#usage-of-keyimporter)
  * [Generated files](#generated-files)
- [Requirements](#requirements)
- [Version Numbers](#version-numbers)
- [Copyright and License](#copyright-and-license)


## Description

A [Bash][bash] script for processing a master GPG keyring and producing a
variety of export files for primary keys and subkeys.

### Usage of `certsplitter`
---
The `certsplitter` command takes the name of a keyring and the prefix of a configuration file to select a pair of PGP certificates. A set of files is created which contain a subset of the subkeys from each of the certificates as well as copies of the certificates themselves.

Each subkey is exported in public and secret versions, and in both GPG and ASCII-armored formats. The primary keys are also exported as a subkey and each certificate is exported in formats for importing into an active keyring and for long-term backup.

The file 'keyring_list' is used to list the keyrings which the system is able to process certificates in. Each keyring is listed as:

    ring_<name>="/path/to/keyring"

where the `<name>` is the name accepted as a command parameter. The `ring_` portion of the variable name is to be used as is. So an entry in the file of `ring_master="/home/user/.keyring"` would be used as 'master' on the command line, like:

    certsplitter master per

Within the keyring list the names `a` thru `z` inclusive are able to be processed by other commands, in the bounds set in the variables `first_ring` and `last_ring`. There is no limit on the size of the list of names, other than the `a` to `z` for the automated system to be developed later. It is also acceptable for nultiple names to have the same path, especially where a 'user-friendly' name is wanted and the associated keyring needs to be in the processing list as well.

The file `.config` is designed to hold data needed to run the script, and to potentially hold sensitive information. The elements of the passphrase can be stored in this file, or the lines for them can be commented and the same variable name can be used to pass them into the program from the environment. As this system is intended to be used in an air-gapped environment, preferably a "live boot" systems, so there is less concern over CLI and environment leakage.

### Usage of `keyimporter`
---

```
keyimporter <ring_name> <config prefix>
keyimporter <ring_name> <config prefix> CERT
keyimporter <ring_name> <config prefix> auth|crypt|sign|ssh [auth|crypt|sign|ssh] ...
```

The `keyimporter` command takes the name of a keyring with the prefix of a configuration file, and optionally a list of purposes and imports the subkeys selected by the configuration file and the listed purposes.

Used with the purpose of `CERT`, with nothing else in the list, will import the full certificate, including the primary key which is intended to _not_ be in the working keyring. Placing `CERT` in a list, first or otherwise, causes an error.

Used with nothing in the purpose list the available subkey for each of the three non-cert purposes will be imported, missing subkeys are silently ignored.

Used with a list, one or more, purposes the subkey for each purpose will be imported, and any missing subkeys will cause an error.

The list of purposes can include, as well, the value `ssh` which will import the SSH data, regardless of whether or not the config name normally would have SSH data imported.

For config prefixes which have `_SSH` or `_GIT` in them, the program will attempt to import the SSH data, and error if the files are not found. The only exception to that is when the purpose list is blank _and_ the auth subkey file is also not available. In silently failing for the auth subkey, the program will bypass the attempt to import the SSH data.

When importing the SSH data, the `sshcontrol` file in the named keyring directory has the key's data appended, and the SSH formated public key is added to the `.ssh` directory within the keyring directory. Under normal use with the `ssh` command these files should be in the user's `$HOME/.ssh` directory and they will need to be moved or copied to that directory after the import is done.


### Generated files
---
The created files are placed into 3 directories, each with its own intended purpose, and content. 

The `imports` directory is for the key, and certificate, extracts of the secret keys. They have passphrases attached which different than the passphrase in the master keyring, and are different between the `key` files and the `cert` files. These files are intended to be imported into the user's working keyring on a regularly connected system. Not all the `key` files will necessarily be imported, only those which are needed on the system. Additionally, unless extreme events occur, the `cert` files should never be imported. Lacking the primary secret key on the working system will prevent modifications to the existing subkeys, or other alterations to the certificates.

The `exports` directory is for the public keys, and certificates, and the files are ready to share, using any method appropriate to such, so that others will have the public half of the keys. Sharing with contacts via email, adding to public key servers, or adding to an account on GitHub are examples of such use. Included in these files are versions suitable for SSH use, including the `sshcontrol` file which has a snippet suitable for adding to the user's same-named file in their `.gnupg` directory.

The `extracts` directory has a copy of the full certificates, with the same passphrase as the master keyring. These files are a backup of the master keyring, and should be kept on the generating system, or in some other secure location. If the master keyring is somehow lost or damaged, these files can be used to replace it.

### Sample files
---
Included in the files are the following:

-   `SAMPLE.cfg`
-   `SAMPLE_CODE.cfg`
-   `SAMPLE_GIT.cfg`
-   `SAMPLE_SSH.cfg`
-   `SAMPLE.keys`
-   `SAMPLE_files.md`

The `*cfg` files are examples of what the config files contain, and were used to create a group of corresponding keys. The generated keys are given in the `SAMPLE.keys` file. Using that keyring and the `*.cfg` files, the files created, with their directories, are listed in the `SAMPLE_files.md`.

[TOP](#contents)

## Requirements

- GnuPG installed on the system
- GNU Bash, v4.3, or better `bash --version | grep version`

_[The program is written in Bash script and uses "Bashisms" which limit its operation in most other Unix shells.]_

[TOP](#contents)

## Version Numbers

gpg-tools uses [Semantic Versioning v2.0.0][semver] as created by [Tom Preston-Werner][tom], inventor of Gravatars and cofounder of GitHub.

Version numbers take the form `X.Y.Z` where `X` is the major version, `Y` is the minor version and `Z` is the patch version. The meaning of the different levels are:

- Major version increases indicates that there is some kind of change in the API (how this program works as seen by the user) or the program features which is incompatible with previous version

- Minor version increases indicates that there is some kind of change in the API (how this program works as seen by the user) or the program features which might be new, while still being compatible with all other versions of the same major version

- Patch version increases indicate that there is some internal change, bug fixes, changes in logic, or other internal changes which do not create any incompatible changes within the same major version, and which do not add any features to the program operations or functionality

[TOP](#contents)

## Copyright and License

The MIT license applies to all the code within this repository.

    Copyright © 2020  Chindraba (Ronald Lamoreaux)
                    <projects@chindraba.work>
    - All Rights Reserved

    Permission is hereby granted, free of charge, to any person
    obtaining a copy of this software and associated documentation
    files (the "Software"), to deal in the Software without
    restriction, including without limitation the rights to use, copy,
    modify, merge, publish, distribute, sublicense, and/or sell copies
    of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be
    included in all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
    EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
    MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
    NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS
    BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN
    ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
    CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.

[TOP](#contents)


  [bash]: https://www.gnu.org/software/bash/
  [semver]: https://semver.org/spec/v2.0.0.html
  [tom]: http://tom.preston-werner.com/
