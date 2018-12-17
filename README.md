# Lolinote Specification

The *Lolinote* is a filesystem-based note-taking data structure. Its philosophy was aimed to extremely simple and intuitive, and let all of the daily usages can be done by human's hands and without dedicated program reasonably. In the same time, still allow external programs to enhance its usability.

This specification defined the data structure of Lolinote 2.0 to allow processing by somewhat programs.



## Specification

The newest specification of Lolinote is [version 2.0-draft]. If you want to find the former specifications, check [Lolinote version 1.0].

[version 2.0-draft]: spec-2.0.md
[Lolinote version 1.0]: https://bitbucket.org/civalin/lolinote/wiki/Home



## Changelog

### 2.0

- enhance: Remove unnecessary restriction to allow the other formats be a note.
- enhance: Remove the restriction of Complex Note, allow it to using sub-directories to storing their attachments.
- simplify: Unify the content's filename and title location of a Complex Note. It will simplify programming, more understandable by intuition, and simplify some tedious maintaining.
- simplify: Re-consider and define which things should be ignoring. Easier for programming.
- change: configuration directory `.loli/` -> `.lolinote/` for improve the explicitness.
