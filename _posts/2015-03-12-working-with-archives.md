---
layout: post
title: Tools For Working With Archives
tags: [zip,unzip,jar]
---

Here I collect some useful tools for working with zip/jar/gz files. As always: just snippets and short descriptions.

## List the contents of an archive

To list the contents of a jar/war/zip archive we type the following:

```bash
jar tf frontend.war
```

## Extract a file

To extract a file from the archive, type something like

```bash
jar xf frontend.war config/app-conf.xml
```

This creates a folder named config with the file app-conf.xml inside. Now you can edit the config file. After you are done editing the file you can update the archive by typing

## Update a file

```bash
jar uf frontend.war config/app-conf.xml
```
## View the contents of a file within the archive

For having a quick look at the contents of a file you can use the unzip command:

```bash
unzip -q -c frontend.war config/app-conf.xml
```

(the c switch redirects output to stdout, the q(uiet) switch suppresses all other output. 


## Test the integrity of archives

```bash
gunzip -t *.gz
```

will report broken gzip archives without unpacking them.