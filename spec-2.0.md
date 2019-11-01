# Specification of Lolinote 2.0-draft-6

> This specification still in draft stage, it maybe minor tweak to fix typo or clarify some part in future.



## Introduction

The *Lolinote* is a filesystem-base note-taking data structure. Its philosophy is aimed to extremely simple and intuitive, and let all of the daily usages can be done by human's hands and without dedicated program reasonably. But in the same time, still allow external programs to enhance its usability.

This specification defined the data structure of Lolinote 2.0 to allow processing by whatever programs.



## Terms & Concepts

### Repository Structures

- *RepositoryRoot*: A directory contain the whole Lolinote Repository
    - *Noise*: Some filesystem entries in a *Repository* but should be ignored and keep it in-place.
    - *Node*: Some filesystem entries in a *Repository* and not a *Noise*. Has following sub-type:
        - *CategoryNode*: A filesystem directory that neighter a *Noise* nor *ComplexNote*.
        - *Note*: A basic unit in a *Repository*. Each notes are independent with the others.
            - *SimpleNote*: Basic type of *Note* (in file form).
            - *ComplexNote*: Advanced type of *Note* (in directory form), support attachments.

Here is a example of *Repository*:

```
any-path-in-filesystem/
    lolinote-root-folder/       # RepositoryRoot
        .lolinote/                  # Noise
        2017-car-review.txt         # SimpleNote
        boardgame-story/            # ComplexNote
            index.html
            1.jpg
        .git/                       # Noise
        .gitignore                  # Noise
        Makefile                    # Noise
```



### Other Terms

- *Filename Extension*: the portion of filename after the last `.`, not include the last `.` itself.
- *Main Filename*: the portion of filename before the last `.`, not include the last `.` itself.



## Details

### Repository

A *Repository* is a subset of filesystem, which started from a *RepositoryRoot*, and ended to some leafs of filesystem entry.

A valid *RepositoryRoot* must have a sub-directory which was named as `.lolinote`. If sub-directory `.lolinote` not exists, the directory cannot be considered as a *RepositoryRoot*. The `.lolinote` directory can be empty or store any data for third-party programs that not be defined in this spec.

One *Repository* can only have one *RepositoryRoot*. If multiple potential *RepositoryRoot* nesting each others, Only single one can be used at the same time.

```
/home/alice/
    travels/                # potential RepositoryRoot #1 ----+
        .lolinote/                                            |
        2019-germany/       # potential RepositoryRoot #2 --+ |
            .lolinote/                                      | |
            plan.txt                                      --+ |
        travel-agencies.txt                               ----+
```

For example, if treat #1 as a *RepositoryRoot* for now, the #2 will just be considered as a normal directory in *Repository* #1 at this time.



### Noise

*Noise* is some filesystem entries which semantic-less in a Lolinote *Repository*. It should be ignored and keep it in-place in this filesystem. Here are the conditions of *Noise*:

1. Files or directories start with `.`. (Core rule 1)
2. Files without *Filename Extension*. (Core rule 2)
3. filesystem entries under *Noise* directory. (recursively)
4. Directories which have more than one file using `index` as those *Main Filename*. (Invalid *ComplexNote*)

Example:

```
/home/alice/
    cooking-notes/
        .lolinote/          # Noise by directory start with `.` (1)
        .git/               # Noise by directory start with `.` (1)
            gitrc           # Noise by recursively (3)
        Makefile            # Noise by file without Filename Extension (2)
        fish/               # Noise by contain two `index` files (4)
            index.md
            index.html
```



### Note

Lolinote spec contain two type of *Note*: *SimpleNote* and *ComplexNote*. No matter which type of *Note*, each *Notes* have the following attributes:

- `title`
- `content_type`
- `content`

All *Notes* are assumed independently with each others. Which mean, any moving or deleting operations can done independently. An example is no link should point to the others notes - because it will cause some link break after moving or deleting.



#### SimpleNote

A *SimpleNote* is just a simple file.

The *Main Filename* is its `title`, the *Filename Extension* is its `content_type`, and the data store in this file is its `content`.

```
lolinote-root-folder/
    .lolinote/
    cooking-recipe.txt      # title: "cooking-recipe", content_type: "txt"
    city-map.jpg            # title: "city-map", content_type: "jpg"
```

Lolinote not defined any `content` format of `content_type`, so all *Filename Extension* were allowed. The true meaning of those `content_type` may or may not be determined by external program or user themself.

Restriction: *SimpleNote* not allow *Main Filename* using the keyword `index`. This is a reserved filename for *ComplexNote*. See below.



#### ComplexNote

A *ComplexNote* is a directory, contain one file directly which *Main Filename* is `index`.

The *ComplexNote*'s `title` is the directory name, the `index`'s *Filename Extension* is its `content_type`, the data store in the `index` file is its `content`. Any other files in this directory (and sub-directories) should be considered as its *Attachments*.

Here is a example of a *ComplexNote*:

```
lolinote-root-folder/
    .lolinote/
    2013-pc-game/       # A complex note entry, title: "2013-pc-game", content_type: "html"
        index.html      # index file (html)
        images/
            1.jpg       # attachment
            2.jpg       # attachment
        music.wav       # attachment
        Makefile        # attachment
```

*ComplexNote* is a "leaf" structure in *Repository* tree. Which mean: any *ComplexNote* will not contain another *Note*. The `1.jpg` and `Makefile` in above example are not a *Note* nor a *Noise* reigned by this *Repository*, they just a attachment -- a part of this *ComplexNote*.

Finally, if any directory contain more than one file which *Main Filename* is `index`, this directory not a *ComplexNote*, just a *Noise*. See the definition of *Noise*.



## The Manual Ordering in a Directory

Every filesystem entries in the same directory should following the ASCII encoding's filename string ordering as their order. In ordering situation, all characters after the first non-ASCII character will be simple ignored.

If two filename has the same ASCII string order, the ordering among them are undefined.

```
lolinote-root-folder/
    .lolinote/
    diary/
        2018-01-12.txt
        2018-01-12-關於新書.txt
        2018-01-13/
            index.md
            1.jpg
        2018-01-25.md
    cooking.txt
    car.txt
```

In above example, `lolinote-root-folder` has 4 entries, the manual ordering should be:

```
.lolinote/
car.txt
cooking.txt
diary/
```

The `diary` directory has 4 entries, the manual ordering is:

```
2018-01-12-關於新書.txt     # "2018-01-12-"    -> "-" == 0x2D in ASCII
2018-01-12.txt              # "2018-01-12.txt" -> "." == 0x2E in ASCII
2018-01-13/
2018-01-25.md
```

Btw, external program may using extra methods to generate or tweak ordering. This section only defined the "manual ordering" assigned by a *Repository* maintainer.
