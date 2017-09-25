---
layout: page
title: Data Management in SD2E
tagline:
---

Data is managed using the `files` service of the Agave CLI. With many parallels in
Linux file commands (`ls, mv, cp, rm,` etc.), Agave `files-*` commands can be used to 
list files, upload and download data, remotely manage and organize data, and import
external data from the web.

Agave `files-*` commands act on Agave `systems`. See the [systems basics](systems_basics.md)
documentation for more details.

This page is broken down into the following sections:

1. [Listing files and navigating](#listing-files-and-navigating)
2. [Uploading and downloading via the CLI](#uploading-and-downloading-via-the-cli)
3. [Other file operations](#other-file-operations)
4. [Share files with another user](#share-files-with-another-user)
5. [Share files using postits](#share-files-using-postits)
6. [Import files from other systems](#import-files-from-other-systems)
7. [Import files from the web](#import-files-from-the-web)


<br>
#### Listing files and navigating

To list the files available to you on the STORAGE system, use:
```
% files-list -S data-tacc-work-username -L /
drwx------  username  4096  13:07  .
```

The `-L` flag provides a long listing (similar to `ls -l` in Linux), and the
forward slash at the end of the line indicates the root path - your `$WORK`
directory.

To make a new folder, then list the contents of that folder::
```
% files-mkdir -S data-tacc-work-username -N sd2e-data /
Successfully created folder sd2e-data

% files-list -S data-tacc-work-username -L /
drwx------  username  4096  15:49  .
drwx------  username  4096  15:49  sd2e-data

% files-list -S data-tacc-work-username -L sd2e-data/
drwx------  username  4096  15:49  .
```

*Tip: if you set `data-tacc-work-username` as your default system, then you 
do not need to provide it on the command line. More info [here](systems_basics.md).*

<br>
#### Uploading and downloading via the CLI

Upload a file from your local system to the STORAGE system with:
```
% touch my_file.txt
% files-upload -S data-tacc-work-username -F my_file.txt sd2e-data/
Uploading my_file.txt...
######################################################################## 100.0%

% files-list -S data-tacc-work-username -L sd2e-data/
drwx------  username  4096  15:53  .
-rw-------  username  0     15:53  my_file.txt
```

Copy the file to something with a new name using the `files-copy` command:
```
% files-copy -S data-tacc-work-username -D sd2e-data/my_file.txt sd2e-data/copy.txt
Successfully copied sd2e-data/my_file.txt to sd2e-data/copy.txt

% files-list -S data-tacc-work-username -L sd2e-data/
drwx------  username  4096  15:57  .
-rw-------  username  0     15:57  copy.txt
-rw-------  username  0     15:53  my_file.txt
```

Then download the result:
```
% files-get -S data-tacc-work-username sd2e-data/copy.txt
######################################################################## 100.0%

% ls
copy.txt  my_file.txt
```

<br>
#### Other file operations

Files can also be moved, deleted, and renamed using the `files-move` and `files-delete`
commands, with similar syntax to the `files-copy` command. For example:
```
% files-move -S data-tacc-work-username -D sd2e-data/file2.txt /sd2e/my_file.txt
% files-mkdir -S data-tacc-work-username -N subfolder/ sd2e-data/
% files-move -S data-tacc-work-username -D sd2e-data/subfolder/file2.txt sd2e-data/file2.txt
% files-delete -S data-tacc-work-username sd2e-data/subfolder/file2.txtt
```

Be cautious with `files-move` and `files-delete` commands. Just like a Linux
filesystem, files inadvertently deleted or written over are probably unrecoverable.

<br>
#### Share files with another user

File permissions are managed similar to Linux file permissions. To list the
permissions on an existing file on the STORAGE system, issue:
```
% files-pems-list -S data-tacc-work-username sd2e-data/my_file.txt
```

The output should look similar to:
```
sd2e READ WRITE EXECUTE
vaughn READ WRITE EXECUTE
username READ WRITE EXECUTE
```

To add permissions for another user to view the file, use the `files-pems-update`
command:
```
% files-pems-update -U my_collaborator -P ALL sd2e-data/my_file.txt
% files-pems-list -S data-tacc-work-username sd2e-data/my_file.txt

my_collaborator READ WRITE EXECUTE
sd2e READ WRITE EXECUTE
vaughn READ WRITE EXECUTE
username READ WRITE EXECUTE
```

Now, a user with username `my_collaborator` has permissions to access the file.
Valid values for setting permission with the `-P` flag are READ, WRITE, EXECUTE,
READ_WRITE, READ_EXECUTE, WRITE_EXECUTE, ALL, and NONE. This same action can be
performed recursively on directories using the `-R` flag.

<br>
#### Share files using postits

Another convenient way to share data is the Agave postits service. Postits
generate a short URL with a user-specified lifetime and limited number of uses.
Anyone with the URL can paste it into a web browser, or curl against it on the
command line. To create a postit: 
```
% postits-create -m 5 -l 3600 https://api.sd2e.org/files/v2/media/system/data-tacc-work-username/sd2e-data/my_file.txt
```

The json response from this command is the URL, e.g.:

``` 
https://api.sd2e.org/postits/v2/866d55b36a459e8098173655e916fa15
```

This postit is only good for 5 downloads (`-m 5`) and only available for one hour (3600 seconds, `-l 3600`). The creator of the postit can list and delete their postits with the following commands:

```
% postits-list -V
% postits-delete 866d55b36a459e8098173655e916fa15
```

The long alphanumeric string is the postit UUID displayed by the verbose postits-list command.

<br>
#### Import files from other systems

It may be useful to import data from other storage systems, e.g. from the community
data space to your private data space. The `files-import` command can be used
for that purpose.

```
% files-import -S data-tacc-work-username \
               -U 'agave://data-sd2e-community/public_data.dat' \
               sd2e-data/
```

With the above syntax, the file located at the root directory on the
`data-sd2e-community` STORAGE system will be imported to your private STORAGE
system, and placed in your directory `sd2e-data/`.

Please also note that even though you are *able* to import files from other
Agave STORAGE systems, you may not always *need* to import those files. Most
applications of Agave will allow you to provide a complete URI path to the file,
e.g. `agave://data-sd2e-community/public_data.dat`. This is useful, for example,
in the case of large reference libraries. Pointing to the remote libraries
rather than copying them saves time and disk space.

<br>
#### Import files from the web

You can also import files from the web using the URL. This is useful
to import files that are not part of an Agave storage system:

```
% files-import -S data-tacc-work-username \
               -U 'https://raw.githubusercontent.com/sd2e/sd2e-cli/master/sd2e-cloud-cli.tgz' \
               -W 'username@email.edu' \
               sd2e-data/
```

The `-W` flag demonstrates a feature that sends an e-mail when the transfer is
complete.

<br>
#### Get help

Reminder: at any time, you can issue an Agave command with the `-h` flag to
find more information on the function and usage of the command. Extensive Agave
documentation can be found [here](http://developer.agaveapi.co/).


---
Return to the [API Documentation Overview](../index.md)
