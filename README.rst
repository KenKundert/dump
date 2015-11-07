dump -- Encrypted Backups to a Remote Server
============================================

Dump is a simple command line utility to orchestrate backups. It is built on 
Duplicity, which is a powerful and flexible utility for managing encrypted 
backups, however it has a rather heavy user interface. With dump, you specify 
all the details about your backups once in advance, and then use a very simple 
command line interface for your day-to-day activities.

Dump is a relatively small python script that designed to be modified to 
customize it for a particular backup. If you wish multiple backups, you simply 
copy the dump script and modify each copy for a particular backup.

Generally, you should dedicate a directory to your backups. That directory will 
hold the dump executable and the archive directory. The archive directory 
contains Duplicity housekeeping files for the backup. You can place as many of 
the dump executables as you wish in that directory and they can be configured to 
share the archive directory.

Backup
------
Once configured, you would perform your first backup as a full backup::

   ./dump backup full

After than, you should prefer incremental backups, which you run using::

   ./dump backup


Manifest
--------

Once a backup has been performed, you can list the files available in your 
backup using::

   ./dump manifest

You can list the files that existed on a particular date using::

   ./dump --date 2015-04-01 manifest

Or, you can list the files that existed 3.5 days ago using::

   ./dump --date 3D12h manifest

The interval string passed as the date is constructed using an integer followed 
by one of the following characters s (seconds), m (minutes), h (hours), 
D (days), W (weeks), M (months), or Y (years). You can combine several to get 
more resolution.


Restore
-------

You restore a file or directory using::

   ./dump restore src/verif/av/manpages/settings.py

Use manifest to determine what path you should specify to identify the desired 
file or directory.

You can restore the version of a file that existed on a particular date using::

   ./dump --date 2015-04-01 restore src/verif/av/manpages/settings.py

Or, you can restore the version that existed 6 months ago using::

   ./dump --date 6M restore src/verif/av/manpages/settings.py


Cleanup
-------

You can clean up your remote repository using::

   ./dump cleanup

This removes any unneeded files Duplicity created files in the remote 
repository.  This is not normally necessary, however it can be helpful if 
a previous session terminated abnormally.

If you also give a date, any backup sets that only contain files that are older 
than the given date are deleted::

   ./dump -d 1Y cleanup

This can reduce the size of your remote repository. Of course after doing this 
you may not be able to restore files that were deleted before the date you 
specified.

Trouble
-------

If Duplicity is refusing to work for you, run using the verbose flags::

   ./dump -v -n backup full

Then carefully read the error messages. They should lead you to the problem.


Installing
----------

You generally would not install dump because the executable is tailored to back 
up a specific directory. Instead you just run it in place. If you would like to 
backup multiple directories, simply make multiple copies of the dump executable 
and customize each one.

Before you can use dump you need to install docopt using::

   yum install python-docopt (or python3-docopt)

You also need `scripts <https://github.com/KenKundert/scripts`_. You can install 
it or simply copy scripts.py into the dump source directory.


Precautions
-----------

You should assure you have a backup copy of the GPG passphrase in a safe place.  
This is very important. If the only copy of the GPG passphrase is on the disk 
being backed up, then if that disk were to fail you would not be able to access 
your backups.

If you keep the GPG passphrase in the dump file, you should set its permissions 
so that it is not readable by others::

   chmod 700 dump

Better yet is to simply not store the passphrase in the dump script. This can be 
arranged if you are using `Abraxas <https://github.com/KenKundert/abraxas>`_, 
which is a flexible password management system. The interface to Abraxas is 
already built in to dump, but its use is optional (it need not be installed).

It is also best, if it can be arranged, to keep your backups at a remote site so 
that your backups do not get destroyed in the same disaster, such as a fire or 
flood, that claims your original files. If you do not have, or do not wish to 
use, your own server, Duplicity offers a number of backends that allow you to 
place your backups in the cloud (Rackspace, Dropbox, Amazon, Google, etc.).  
Remember, your data is fully encrypted, so they cannot pry.


Duplicity
---------
Between Duplicity version 0.6.25 and 0.7.05 the way you specify the SSH backend 
changes. Duplicity provides several different implementations of the SSH 
backend. The default is paramiko, however it does not support bandwidth 
limiting. So instead, dump uses the pexpect version. In version 0.6.25 the 
backend was specified with '--ssh-backend pexpect'. In version 0.7.05 it is now 
specified by adding it to the protocol specification for the remote destination, 
so 'sftp://...' changes to 'pexpect+sftp://...'.

To address this, dump provides the SSH_BACKEND_METHOD which should be set to 
'option' for Duplicity version 0.6.25 and lower, and should be set to 'protocol' 
for version 0.7.05 and above.
