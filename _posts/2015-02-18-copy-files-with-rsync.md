---
layout: post
title: Sync A Folder Between Two Machines
tags: [rsync]
---


To sync a folder between two machines you can use rsync. The following command will copy the contents of the folder *source* to the folder *target-folder* on the remote host using ssh:

---

```rsync -avz source user@host:target-folder```

This command uses the following options:

* -a, --archive 

    This is equivalent to -rlptgoD. It is a quick way  of  saying  you  want  recursion  and  want  to preserve  almost  everything  (with -H being a notable omission).  The only exception to the above equivalence is when --files-from is specified, in which case -r is not implied.

    Note that -a does not preserve hardlinks, because finding multiply-linked files is expensive.  You must separately specify -H.

* -v, --verbose       

    This  option  increases  the amount of information you are given during the transfer.  By default, rsync works silently. A single -v will give you information about what files are being transferred and  a  brief summary at the end. Two -v options will give you information on what files are being skipped and slightly more information at the end. More than two -v options should only be used  ifyou are debugging rsync.

    Note that the names of the transferred files that are output are done using a default --out-format of "%n%L", which tells you just the name of the file and, if the item is a link, where it  points. At  the  single  -v  level  of  verbosity,  this  does not mention when a file gets its attributes changed.  If you ask for an itemized list  of  changed  attributes  (either  --itemize-changes  or adding  "%i"  to  the  --out-format  setting), the output (on the client) increases to mention all items that are changed in any way.  See the --out-format option for more details.    

* -z, --compress

    With  this  option, rsync compresses the file data as it is sent to the destination machine, which reduces the amount of data being transmitted -- something that is useful over a slow connection.

    Note that this option typically achieves better compression ratios than can be achieved by using a compressing  remote  shell  or  a compressing transport because it takes advantage of the implicit information in the matching data blocks that are not explicitly sent over the connection.

    See the --skip-compress option for the default list of file suffixes that will not be compressed.


The description of the parameters is taken from [explainshell](http://explainshell.com/explain?cmd=rsync+-avz+source-folder+user%40target-machine%3Atarget-folder).