<code>find_usbdev</code> - more on this later.

<code>recfixsymlink</code> - Recursively fix symlinks in the directory from which the script is run

<pre>
$ recfixsym --help

Purpose: recursively fix symlinks in the directory tree relative to your PWD, which point
         to an invalid file name and replace the erroneous symlinks to point to a correct
         endpoint. USE-CASE: you change a filename to which 20 symlinks pointed. Provided
         the symlinks are in the directory tree you're currently in, this tool will fix
         the symlinks to point to the filename you just changed.

Usage: /home/jimconn/bin/recfixsym <--relink to_be_fixed> <--to file_to_link_to> [--as new_symlink_name] --help

** This must run from the directory containing the file to which you are relinking **

--relink <old symlink>     : the symlink in recursively found directories to be changed.
--to <file_to_link>        : the file to which the new symlink(s) will point.
--as <filename>            : make the new symlink filename <filename>
--help - this help message : the file to which you're new symlink will be created.
</pre>

Here's a full directory tree. Notice the symlinks, because that's where I want to focus on. Notice the symlinks, below, called <code>init_vars.tf</code>.

<pre>
$ ls -lR
...
./instances/ramfs:
total 12
drwxrwxr-x 7 jimconn jimconn 4096 Dec  1 14:33 ..
-rw-rw-r-- 1 jimconn jimconn  778 Dec  1 17:35 instances.tf
lrwxrwxrwx 1 jimconn jimconn   27 Dec  2 11:04 init_vars.tf -> /var/tmp/devel/init_vars.tf
lrwxrwxrwx 1 jimconn jimconn   31 Dec  2 11:04 terraform.tfvars -> /var/tmp/devel/terraform.tfvars
drwxrwxr-x 2 jimconn jimconn 4096 Dec  2 11:04 .

./instances/master-ccn:
total 16
-rw-rw-r-- 1 jimconn jimconn   35 Dec  1 14:33 README.md
drwxrwxr-x 7 jimconn jimconn 4096 Dec  1 14:33 ..
-rw-rw-r-- 1 jimconn jimconn  772 Dec  1 17:34 instances.tf
lrwxrwxrwx 1 jimconn jimconn   27 Dec  2 11:04 init_vars.tf -> /var/tmp/devel/init_vars.tf
lrwxrwxrwx 1 jimconn jimconn   31 Dec  2 11:04 terraform.tfvars -> /var/tmp/devel/terraform.tfvars
drwxrwxr-x 2 jimconn jimconn 4096 Dec  2 11:04 .
...
</pre>

And a little more localized; let's start here. Notice above in the child directories where the symlinks are pointing to files. The init_vars.tf links are pointing to
the <code>init_vars.tf</code> file here:
<pre>
$ ls -altr
total 52
drwxrwxr-x  2 jimconn jimconn  4096 Dec  1 12:46 data-persistence
-rw-rw-r--  1 jimconn jimconn   235 Dec  1 14:33 terraform.tfvars
drwxrwxr-x  7 jimconn jimconn  4096 Dec  1 14:33 instances
-rw-rw-r--  1 jimconn jimconn     0 Dec  1 18:48 outputs.tf
-rw-rw-r--  1 jimconn jimconn  1989 Dec  2 08:46 README.md
-r--r--r--  1 jimconn jimconn   293 Dec  2 09:26 init_vars.tf
drwxrwxr-x  8 jimconn jimconn  4096 Dec  2 10:26 .
drwxrwxr-x  2 jimconn jimconn  4096 Dec  2 10:27 vpc
drwxrwxr-x  2 jimconn jimconn  4096 Dec  2 10:27 storage
drwxrwxr-x  2 jimconn jimconn  4096 Dec  2 10:27 global
drwxrwxr-x  2 jimconn jimconn  4096 Dec  2 10:27 monitoring
drwxrwxrwt 31 root    root    12288 Dec  2 10:46 ..
</pre>

I want to change the name of the init_vars.tf file to <code>variables.tf</code>, but if I do that, all of the symlinked files linked to this file will no longer be valid.
This script, <code>recfixsym</code> can fix this problem.

<pre>
$ mv init_vars.tf variables.tf
$ ls -ltr
total 36
drwxrwxr-x 2 jimconn jimconn 4096 Dec  1 12:46 data-persistence
-rw-rw-r-- 1 jimconn jimconn  235 Dec  1 14:33 terraform.tfvars
drwxrwxr-x 7 jimconn jimconn 4096 Dec  1 14:33 instances
-rw-rw-r-- 1 jimconn jimconn    0 Dec  1 18:48 outputs.tf
-rw-rw-r-- 1 jimconn jimconn 1989 Dec  2 08:46 README.md
-r--r--r-- 1 jimconn jimconn  293 Dec  2 09:26 variables.tf
drwxrwxr-x 2 jimconn jimconn 4096 Dec  2 11:04 global
drwxrwxr-x 2 jimconn jimconn 4096 Dec  2 11:04 storage
drwxrwxr-x 2 jimconn jimconn 4096 Dec  2 11:04 monitoring
drwxrwxr-x 2 jimconn jimconn 4096 Dec  2 11:04 vpc
</pre>

So, <code>init_vars.tf</code> NO LONGER exists. Thus:

<pre>
$ ls -ltrR
...
./instances/ramfs:
total 12
drwxrwxr-x 7 jimconn jimconn 4096 Dec  1 14:33 ..
-rw-rw-r-- 1 jimconn jimconn  778 Dec  1 17:35 instances.tf
lrwxrwxrwx 1 jimconn jimconn   27 Dec  2 11:04 init_vars.tf -> /var/tmp/devel/init_vars.tf           << -- no longer valid
lrwxrwxrwx 1 jimconn jimconn   31 Dec  2 11:04 terraform.tfvars -> /var/tmp/devel/terraform.tfvars
drwxrwxr-x 2 jimconn jimconn 4096 Dec  2 11:04 .

./instances/master-ccn:
total 16
-rw-rw-r-- 1 jimconn jimconn   35 Dec  1 14:33 README.md
drwxrwxr-x 7 jimconn jimconn 4096 Dec  1 14:33 ..
-rw-rw-r-- 1 jimconn jimconn  772 Dec  1 17:34 instances.tf
lrwxrwxrwx 1 jimconn jimconn   27 Dec  2 11:04 init_vars.tf -> /var/tmp/devel/init_vars.tf           << -- no longer valid
lrwxrwxrwx 1 jimconn jimconn   31 Dec  2 11:04 terraform.tfvars -> /var/tmp/devel/terraform.tfvars
drwxrwxr-x 2 jimconn jimconn 4096 Dec  2 11:04 .
...
</pre>

And normally, to fix this, you'd have to write some kind of one-liner or script or come up with some other means to fix this issue. I wrote <code>recfixsym</code> and
decided to put it out there.

<pre>
$ recfixsym --relink init_vars.tf --to variables.tf
...
./instances/ramfs:
total 4
-rw-rw-r-- 1 jimconn jimconn 778 Dec  1 17:35 instances.tf
lrwxrwxrwx 1 jimconn jimconn  31 Dec  2 11:04 terraform.tfvars -> /var/tmp/devel/terraform.tfvars
lrwxrwxrwx 1 jimconn jimconn  27 Dec  2 11:06 variables.tf -> /var/tmp/devel/variables.tf            << -- valid again

./instances/master-ccn:
total 8
-rw-rw-r-- 1 jimconn jimconn  35 Dec  1 14:33 README.md
-rw-rw-r-- 1 jimconn jimconn 772 Dec  1 17:34 instances.tf
lrwxrwxrwx 1 jimconn jimconn  31 Dec  2 11:04 terraform.tfvars -> /var/tmp/devel/terraform.tfvars
lrwxrwxrwx 1 jimconn jimconn  27 Dec  2 11:06 variables.tf -> /var/tmp/devel/variables.tf            << -- valid again
...
</pre>

Now one can see that the issue that existed before is now fixed with one command.
