DUMP: Encrypted Backups to a Remote Server
==========================================

Dump is a simple command line utility to orchestrate backups. It is based on 
duplicity, which is a powerful and flexible utility for managing encrypted 
backups, however it has a rather heavy user interface. With dump, you specify 
all the details about your backups once in advance, and then use a very simple 
command line interface for your day to day activities.

Dump is a relatively small python script that designed to be modified to 
customize it for a particular backup. If you wish multiple backups, you simply 
copy the dump script and modify each copy for a particular backup.

Generally, you should dedicate a directory to your backups. That directory will 
hold the dump executable and the archive directory. The archive directory 
contains duplicity housekeeping files for the backup. You can place as many of 
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

   ./dump --date -3D12h manifest

The interval string can be constructed using the characters s (seconds), 
m (minutes), h (hours), D (days), W (weeks), M (months), or Y (years).


Restore
-------

You restore a file or directory using::

   ./dump restore src/verif/av/manpages/settings.py

Use manifest to determine what path you should specify to identify the desired 
file or directory.

You can restore the version of a file that existed on a particular date using::

   ./dump --date 2015-04-01 restore src/verif/av/manpages/settings.py

Or, you can restore the version that existed 6 months ago using::

   ./dump --date -6M restore src/verif/av/manpages/settings.py


Cleanup
-------

You can clean up your remote repository using::

   ./dump cleanup

This removes any unneeded files duplicity created files in the remote 
repository.  This is not normally necessary, however it can be helpful if 
a previous session terminate abnormally.

If you also give a date, any backup sets that only contain files that are older 
than the given date are deleted::

   ./dump -d 1Y cleanup


Trouble
-------

If duplicity is refusing to work for you, run using the verbose flags::

   ./dump -v -n backup full

And carefully read the error messages. They should lead you to the problem.


Installing
----------

This program is not generally installed. Instead, it is placed in the backups 
directory, customized to fit the situation, and used in place. It requires the 
use of docopt, a standard python package that can be installed using pip. It 
also requires script, which can be downloaded from `Github 
<https://github.com/KenKundert/scripts>`.


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
arranged if you are using `Abraxas <https://github.com/KenKundert/abraxas>`, 
which is a flexible password management system. The interface to Abraxas is 
already built in to dump, but its use is optional (it need not be installed).

It is also best, if it can be arranged, to keep your backups at a remote site so 
that your backups do not get destroyed in the same disaster, such as a fire or 
flood, that claims your original file. Duplicity offers a number of backends 
that allow you to place your backups in the cloud (Rackspace, Dropbox, Amazon, 
Google, etc.).  Remember, your data is fully encrypted, so they cannot pry.
