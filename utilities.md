UNIX utilities
==============

A large number of utility programs are installed on most UNIX systems.
Most of these are developed and maintain as part of the [GNU core
utilities](https://www.gnu.org/software/coreutils/coreutils.html).
Many of these utilities are installed in the directory `/usr/bin`.  We
will discuss just a few of them here.  A longer list of tools with
documentation is
[here](http://www.tldp.org/LDP/GNU-Linux-Tools-Summary/html/book1.htm).

Data transfer
-------------

[wget](https://www.gnu.org/software/wget/manual/html_node/index.html) ("web get")
is a tool for obtaining files from web servers
using the command line.  The common usage is to pass wget a url, e.g.

```
> wget https://en.wikipedia.org/wiki/Wget
```

[scp](http://tldp.org/LDP/GNU-Linux-Tools-Summary/html/x9094.htm)
 ("secure copy") transfers files between computers, and handles
 login authentication when needed.  The basic pattern is `scp from
 to`, where *from* and *to* specify some combination of the user, the
 host, and the file name.  In the example below, we copy a file named
 *filename* from an account on the machine *login.itd.umich.edu* to
 the current working directory.  The user will be prompted to
 authenticate on the remote machine after issuing this command.

```
> scp kshedden@login.itd.umich.edu:filename .
```

Here is an *scp* command that transfers a file from the local machine
to *login.itd.umich.edu*:

```
scp filename kshedden@login.itd.umich.edu.
```

Compression and archiving of files
----------------------------------

The most common compression format is probably
[gzip](https://en.wikipedia.org/wiki/Gzip).  To compress one or more
files use `gzip <files>`, and to decompress them use `gunzip <files>`.
Other compression formats/utilities include `bzip2`, `xz`, and the
Windows-compatible `zip`/`unzip`.

Gzip compressed files should have names ending in *.gz*.  Many tools
such as emacs and less can read compressed files without decompressing
on-disk, i.e. if you run `less file.gz` you will see the decompressed
contents of the file.  Some tools such as grep provide an alternate
version (`zgrep`) for handling compressed inputs.  Many programming
languages can also read compressed files directly without on-disk
decompression.  In general, once you compress a file using gzip you
should not have to decompress it ever again.

If a command line tool does not support gzip files directly, you can
use a pipe to decompress the file on-the-fly.  For example, if you
want to know how many lines of the compressed file *filename.gz*
contain the text string *term*, you can use:

```
> gzip -dc <filename.gz> | grep <term> | wc -l
```

*Archiving* creates a single file that contains the contents of many
 files.  It is a reversible process, so the archive can, say, be
 transferred to another machine and then opened up.  The UNIX archive
 utility is called `tar`.  Use `tar -xvf files.tar` to restore the
 individual files, and use `tar -cvf archive.tar file1 file2...` to
 create an archive.

Working with text files
-----------------------

Here we cover some basic tools for working with text files, more advanced
tools like sed and awk are covered in a separate section.  Note that most
of these utilities operate on "data streams", meaning that the data
are incrementally transformed rather then being read all at once and
transformed as a chunk.  This allows these tools to work with data files
that are much larger than the memory.  Note that a few processing algorithms
like sorting cannot be carried out in a streaming manner, but there are still
work-arounds in the sort implementation that allow it to work on files
that are larger than memory.


[cat](https://www.gnu.org/software/coreutils/manual/html_node/cat-invocation.html#cat-invocation) ("concatenate")
 combines the contents of several text files
 sequentially.  By default, the results go to the screen ("standard
 output"), but typically this will be redirected to a file:

```
> cat file1 file2 > result
```

[grep](https://www.gnu.org/software/grep/manual/grep.html)
 is used to search text files.  It has many features and
 options.  The most basic usage is `grep <term> <file>`, which returns
 all lines of the file named *file* that contain the text term *term*.


[wc](https://www.gnu.org/software/coreutils/manual/html_node/wc-invocation.html) ("word count")
 is used to count words or lines in a text file.  A
 basic usage is `wc -l filename` to count the lines in a file.  It is
 often used together with grep to determine how many lines match a
 given pattern, e.g.

```
> grep word files | wc -l
```

[sort](https://www.gnu.org/software/coreutils/manual/html_node/sort-invocation.html)
takes the lines of all input files together and sorts them.  A
basic usage would be `sort file1 file2 > output`.  It is capable of
sorting very large files out-of-memory.  There are many options, for
example, if we have a csv file we can sort based on the third column
using the comma character as a delimiter:

```
> sort -k3,3 -t ',' file
```

Note that this is an [external sort](https://en.wikipedia.org/wiki/External_sorting)
that never needs to read an entire file into memory at once.

[paste](https://www.gnu.org/software/coreutils/manual/html_node/paste-invocation.html)
 combines multiple input files horizontally (as opposed to
 `cat` which combines files vertically).  Basic usage is `paste file1
 file2 > output`.

[cut](https://www.gnu.org/software/coreutils/manual/html_node/The-cut-command.html)
is mainly used for extracting specific fields from a delimited file.  For a CSV
file, `cut -d, -f2 file.csv` would extract the second field, using a comma as the
delimiter.

[head](https://www.gnu.org/software/coreutils/manual/html_node/head-invocation.html) and
[tail](https://www.gnu.org/software/coreutils/manual/html_node/tail-invocation.html)
 extract the first or last 10 lines of a file and
 write them to standard output (screen).  The number of lines is
 configurable using the `-n` flag, e.g.

```
> head -n50 file.csv
```

[join](https://www.gnu.org/software/coreutils/manual/html_node/join-invocation.html#join-invocation)
 merges (joins) two files on a common key.  Both of the files being merged must be sorted before
 invoking `join`.  The join is an "outer join" - all combinations of matching keys are included
 in the result.

 ```
> sort -k 7 file1.txt > file1_sorted.txt
> sort -k 3 file2.txt > file2_sorted.txt
> join -1 7 -2 3 file1_sorted.txt file2_sorted.txt > joined.txt
 ```

[comm](https://www.gnu.org/software/coreutils/manual/html_node/comm-invocation.html)
identifies mathing lines in two (pre-sorted) files.  By default, the output has three
columns: the first column contains lines unique to file 1, the second column contains
files unique to file 2, and the third column contains lines present in both files.

```
> comm file1.txt file2.txt
> comm -12 file1.txt file2.txt # Only print lines that are in both files
> comm -2 file1.txt file2.txt # Only print lines that are unique to file 2
```

