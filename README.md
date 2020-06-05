# IMAPdedup
*A duplicate email message remover*

**Please note that recent versions require Python 3.6 or later**

IMAPdedup is a Python script (imapdedup.py) that looks for duplicate messages in a set of IMAP mailboxes and tidies up all but the first copy of any duplicates found. 

To be more exact, it *marks* the second and later occurrences of a message as 'deleted' on the server.   Exactly what that does in your environment will depend on your mail server and your mail client. 

Some mail clients will let you still view such messages *in situ*, so you can take a look at what's happened before 'compacting' the mailbox.  Sometimes deleted messages appear in a 'Trash' folder.  Sometimes they are hidden and can be displayed and un-deleted if wanted, until they are purged.   

Whatever your system does, you will usually have the option to see what has been deleted, and to recover it if needed, using your email program, after running this script.  (If your server purges the deleted messages automatically, you may be able to prevent this with the *--no-close* option.)

## How it works

By default, IMAPdedup will simply look for messages with a duplicate Message-ID header.  This is a string generated by email systems that should normally be unique for any given message, so unless you've got some rather unusual mailboxes, it's a pretty safe choice.  (Note that GMail, for example, *does* sometimes have some unusual mailboxes.)

If you have messages *without* a Message-ID header, or you don't trust it, there's an option (-c) to use a checksum of the To, From, Subject, Date, Cc & Bcc fields instead.

And if you want to add the Message-ID, if it exists, into this checksum, add the '-m' option as well. I'd recommend this in general, because some (foolish) automated systems can send you multiple messages within a single second, with different contents but the same headers. (e.g. "Subject: Your review has just been published!")

## Installation

IMAPdedup doesn't currently have any installation process.  You just need the imapdedup.py file, and Python 3.

## Trying it out

If you want to experiment, create a new folder on your mail server, and copy some messages into it from your inbox.  Then copy some of them in a second time.  And maybe a third. That should give you a safe place to play!

## Simple use

*Note: IMAPdedup expects to run under Python 3.*

You can list the full syntax by running:

    ./imapdedup.py -h

but the key options are described below.  You will of course need the address of your IMAP email server, and your username on that server.

If you're not on a system that can execute Python scripts directly like this, you may need something like:

    python3 imapdedup.py -h

Try starting with something harmless like:

    ./imapdedup.py -s imap.myisp.com -u myuserid -x -l

which prompts you for your password and then lists the mailboxes on the server. You can then use the mailbox names it returns when running other commands. (The `-x` option specifies that the connection should use SSL, which is generally the case nowadays. If this doesn't work, you can leave it out, but you should probably also complain to your email provider because they aren't providing sufficient security! A future version of the script will make this the default.)

It's worth trying getting this list at least once because different mail servers structure their folders differently. One of mine thinks of all the folders as being 'within' the inbox, for example, so they're called things like 'INBOX.Drafts','INBOX.Sent', and those are the names I need to use when talking to the server.

Once you know your folder names, you can run something like

    ./imapdedup.py -s imap.myisp.com -u myuserid -x -n INBOX.Test

and the script will tell you what it would do to your *INBOX/Test* folder.  If your folder name contains spaces, you'll need to put it in quotes:

    ./imapdedup.py -s imap.myisp.com -u myuserid -x -n "My Important Messages"

The `-n` option tells IMAPdedup that this is a 'dry run': it stops it from *actually making* any changes; it's a good idea to run with this first unless you like living dangerously.  When you're ready, leave that out, and it will go ahead and mark your duplicate messages as deleted.  

The process can take some time on large folders or slow connections, so you may want to add the `-v` option to give you more information on how it's progressing.

You can specify multiple folders to work on, and it work through them in order and will delete, in the later folders, duplicates of messages that it has found either in those folders or in earlier ones.

# Specifying the password

If you don't wish to specify a password via a command-line argument, where it could be seen by other users of the system, and you don't want to type it in each time, you have two options:

* You can put it in an environment variable called IMAPDEDUP_PASSWORD, or
* You can specify it in a wrapper script as described below.

# Use with a config file (a wrapper script)

Michael Haggerty made some small changes to facilitate calling imapdedup from a script (e.g., from a cron job).  Instead of running it directly, create a wrapper script that can be as simple as:

    #! /usr/bin/env python

    import imapdedup

    class options:
        server = 'imap.example.com'
        port = None
        ssl = True
        user = 'me'
        password = 'Pa$$w0rd'
        verbose = False
        dry_run = False
        use_checksum = False
        use_id_in_checksum = False
        just_list = False
        no_close = False
        process = False

    mboxes = [
        'INBOX',
        'Some other mailbox',
        ]

    imapdedup.process(options, mboxes)

Note that you will normally need to include in your options class **ALL of the options** that you might specify on the command line.  
If new options are added to the main imapdedup.py script, you'll need to update your wrapper script to specify them.

If you on a shared machine or filesystem and you are including sensitive information such as the password in this file, you may wish to set its permissions appropriately.

## Accessing the IMAP mailboxes via a local server

The -P option allows you to access the mailboxes via stdin/stdout to a subprocess, rather than over the network.
Dovecot can be run in this mode, for example:

    /usr/lib/dovecot/imap -o mail_location=maildir:~/.mbsync/mails

Typically you might wrap such a command in a script, and then specify the script as the argument of the -P option.


## Acknowledgements etc

For more information, please see [the page on Quentin's site](https://quentinsf.com/software/imapdedup).

This software is released under the terms of the GPL v2.  See the included LICENCE.TXT for details.  

It comes with no warranties, express or implied; use at your own risk!

Many thanks to Liyu (Luke) Liu, Adam Horner, Michael Haggerty, 'GargaBou', Stefan Agner, Vincent Bernat, Jonathan Vanasco and others for their contributions!

[Quentin Stafford-Fraser][1]

[1]:https://statusq.org

