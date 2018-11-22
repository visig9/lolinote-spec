# Specification of Lolinote 2.0-draft-2

> This specification still in draft stage, it maybe minor tweak to fix typo or extend some part in future.



## Introduction

The *Lolinote* is a filesystem-base note-taking data structure. Its philosophy was aimed to extremely simple and intuitive, and let all of the daily usages can be done by human's hands and without dedicated program reasonably. But, in the same time, still allow external programs to enhance its usability.

This specification defined the data structure of Lolinote 2.0 to allow processing by any programs.



## Basic Terms & Concepts

- *Lolinote Instance*: The largest unit of Lolinote specification.
- *Instance Root*: A specific directory contain a whole *Lolinote Instance*.
- *Instance Configuration Folder*: A specific directory named `.lolinote` under the *Lolinote Instance Root*.
- *Note*: A basic unit in a *Lolinote Instance*. Each notes are independent with the others.
    - *Simple Note*: Basic type of *Note* (in file form).
    - *Complex Note*: Advanced type of *Note* (in directory form), support attachments.
- *Noise*: Some filesystem entries exist in Lolinote Instance but should be ignored and keep it in-place.
- *Filename Extension*: the portion of filename after the last `.`.
- *Main Filename*: the portion of filename before the last `.`.

Here is a example of a *Lolinote Instance*:

```
any-path-in-filesystem/
    lolinote-root-folder/           # Instance Root
        .lolinote/                  # Instance Configuration Folder
        2017-car-review.txt         # a Simple Note - filename extension: txt, main filename: 2017-car-review
        boardgame-story/            # a Complex Note
            index.html
            1.jpg
        .git/                       # Noise
        .gitignore                  # Noise
        MakeFile                    # Noise
```



## Detail of Definitions

### Lolinote Instance

A *Lolinote Instance* begin with a directory (called *Instance Root*) in any locations of a filesystem. A *Instance Root* must have a sub-directory which was named `.lolinote` (called *Instance Configuration Folder*).

In other words, any directories which contain a folder named `.lolinote` directly should be considered as a independent *Lolinote Instance*.

An example:

```
/home/alice/
    cooking-notes/          # the root directory of a lolinote instance
        .lolinote/
    travels/
        germany/            # the root directory of another lolinote instance
            .lolinote/
        french/             # the root directory of yet another lolinote instance
            .lolinote/
```

A *Lolinote Instance* including all of the recursive sub-directories and files.



### Noise

*Noise* is somethings which Lolinote program should ignore and keep it in-place in a *Lolinote Instance*. Including:

1. Any files without *Filename Extension*.
2. Any files or directories start with `.`.
3. Any other *Lolinote Instance*.
4. Any directories contain multiple files which *Main Filenames* is `index`.
5. Any files or directories under a *Noise* element.

An example:

```
/home/alice/
    cooking-notes/
        .lolinote/
        MakeFile                    # Noise: without filename extension (1)
        .git/                       # Noise: start with `.` (2)
        other-note-instance/        # Noise: Other Lolinote Instance (3)
            .lolinote/
```



### Note

Lolinote contain two type of Note: *Simple Note* and *Complex Note*. No matter which type of *Note*, each *Notes* have the following data fields:

- `title`
- `content_type`
- `content`



#### Simple Note

A *Simple Note* is just a simple file. The *Main Filename* is its `title`, the *Filename Extension* is its `content_type`, the data store in this file is its `content`.

Here is a example:

```
lolinote-root-folder/
    .lolinote/
    cooking-recipe.txt      # This is a valid note. title: "cooking-recipe", content_type: "txt"
```

Lolinote not defined the `content` format of each `content_type`, so, any *Filename Extension* were allowed. The true meaning of those `content_type` may or may not be determined by external program or user themself.

```
lolinote-root-folder/
    .lolinote/
    city-map.jpg            # This is a valid note. title: "city-map", content_type: "jpg"
```

Here is a restriction: *Simple Note* not allow the *Main Filename* use the keyword `index`. This is a reserve word for *Complex Note*. See below.



#### Complex Note

A *Complex Note* is a directory, contain a file which *Main Filename* is `index`.

The *Complex Note*'s `title` is the directory name, the index's *Filename Extension* is its `content_type`, the data store in the index file is its `content`. Any other files in this directory (and sub-directories) should be considered as the *Attachments* of this *Note*.

Here is a example of a *Complex Note*:

```
lolinote-root-folder/
    .lolinote/
    2013-pc-game/       # A complex note entry, title: "2013-pc-game", content_type: "html"
        index.html      # index file
        images/
            1.jpg       # attachment
            2.jpg       # attachment
        music.wav       # attachment
        MakeFile        # attachment
```

A *Note* is a "leaf" structure in *Lolinote Instance* tree. Which mean: any *Notes* will not contain another *Note*. The `1.jpg` and `MakeFile` in above example are not a *Note* or a *Noise* reigned by this *Lolinote Instance*, they just a attachment -- a part of this *Complex Note*.

Finally, if any directory contain more than one file which *Main Filename* is `index`, this directory not a *Complex Note*, just a *Noise*. See the definition of *Noise*.



## The Manual Ordering in a Directory

Every filesystem entries in the same directory should following the ASCII encoding's filename string ordering as their order. In ordering situation, all characters after the first non-ASCII character will be simple ignored.

If two filename has the same ASCII string order, the ordering among them are undefined.

Program can using more methods to generate or tweak the ordering. This protocol are only defined the manual order assigned by a Lolinote Instance maintainer.

Here is a example:

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

In the above example, `lolinote-root-folder` has 4 entries, the manual ordering should be:

```
.lolinote/
car.txt
cooking.txt
diary/
```

The `diary` directory has 4 entries, the manual ordering is:

```
2018-01-12-關於新書.txt     # only consider "2018-01-12-"       "-" == 0x2D in ASCII
2018-01-12.txt              # consider      "2018-01-12.txt"    "." == 0x2E in ASCII
2018-01-13/                 # consider      "2018-01-13"
2018-01-25.md
```



## Instance Configuration Folder

The *Instance Configuration Folder* (`.lolinote/`) can be empty, or used to store any lolinote's program related data. Those program related data should only be saved under a sub-directory named with the "program name". For example:

```
lolinote-root-folder/
    .lolinote/
        lolinote-data-exportor/     # program 1
            config.ini
            cache.json
        static-website-builder/     # program 2
            conf.yaml
            build/
                index.html
                style.css
        lolisearch/                 # program 3
            reverse-index.json
```

This specification wan't define what is the "program name". But it should be a human readable name and can easily to use to find the related project.