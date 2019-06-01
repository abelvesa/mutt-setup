# mutt setup

This setup guide is made to allow mutt to work with Microsoft Exchange Server
allowing you to switch from Outlook to mutt.

Note that this setup uses offlineimap which implies that all the emails will
be downloaded to the workstation.

The way this works is as follows:
 - [davmail](http://davmail.sourceforge.net/) acts as gateway to the MS Exchange Server
 - [msmtp](https://marlam.de/msmtp/) uses davmail to send emails
 - [offlineimap](http://www.offlineimap.org/) uses davmail to download emails
 - [neomutt](https://neomutt.org/) is configured to make use of the local mailbox, downloaded by offlineimap, and msmtp for sending

Also, note that this guide has been written for Ubuntu and tested on 16.04
and 18.04, but the same steps apply to any other distro, the differences
being in what package manager is your distro using and (maybe) the availability
of the dependencies.

The order you set every component up in doesn't necessarily have to be as
bellow.

1. [ mutt (neomutt) ](#mutt)

2. [ davmail ](#davmail)

3. [ offlineimap ](#offlineimap)

4. [ msmtp ](#msmtp)

5. [ mailcap ](#mailcap)

6. [ todos ](#todos)

<a name="mutt"></a>
## 1. mutt (neomutt)

The build dependencies you need to install are the following:

```console
 $ sudo apt install build-essential xsltproc libxml2-utils libncursesw5-dev libidn11-dev libkyotocabinet-dev
```

Now clone, build and install it:
```console
 $ git clone https://github.com/neomutt/neomutt
 $ cd ./neomutt
 $ ./configure --disable-doc --disable-nls --kyotocabinet
 $ make && sudo make install
```

Make sure the ~/.mail dir exists:
```console
 $ mkdir ~/.mail
```

Copy the dot files from the mutt-setup to their rightful location:
```console
 $ cp ./mutt-setup/dotfiles/.muttrc ~/.
 $ cp ./mutt-setup/dotfiles/.mailboxes ~/.mail/.
 $ cp ./mutt-setup/dotfiles/.aliases ~/.mail/.
```

The .mailboxes file tells mutt which folder names to load in the sidebar.
You'll have to set up the list with your own maliboxes.

The .aliases file holds entries of contacts for quick access.
You'll also have to write your own aliases. Mutt has a shortcut that allows
you to save the sender of an email to the aliases file.

Edit the 'from' entry in ~/.muttrc and set it to your email.
You can also tweak the .muttrc to match your needs.

<a name="davmail"></a>
## 2. davmail

The build dependencies you need to install are the following:
```console
 $ sudo apt install ant
```

Now clone and build:
```console
 $ git clone https://github.com/mguessan/davmail.git
 $ cd ./davmail
 $ export ANT_OPTS=-Dfile.encoding=UTF-8
 $ ant
```

Copy the .davmail.properties to home dir:
```console
 $ cp ./mutt-setup/dotfiles/.davmail.properties ~/.
```

Setup the davmail service:
```console
 $ sudo cp ./mutt-setup/systemd/davmail.service /etc/systemd/system/.
```

Edit the /etc/systemd/system/davmail.service and add the path
to the davmail git repo.

Once you're done with that you can enable and start the service.
```console
 $ sudo systemctl enable davmail.service
 $ sudo systemctl start davmail.service
```

<a name="offlineimap"></a>
## 3. offlineimap

The build dependencies you need to install are the following:
```console
 $ sudo apt-get install python-minimal python-six python-imaplib2
```

Clone it:
```console
 $ git clone https://github.com/OfflineIMAP/offlineimap.git
```

Copy the .offlineimaprc to home dir:
```console
 $ cp ./mutt-setup/dotfiles/.offlineimaprc ~/.
```

Edit the ~/.offlineimaprc and replace the email and the password
with your own.

The initial sync will take a long time and the mailbox will use a
lot of space. Again, offlineimap will download the entire mailbox
to your workstation. You can configure which mailbox folder in
your ~/.offlineimaprc to be synced by using a folder filter like
this:
```console
folderfilter = lambda folder: folder not in [ 'Spam'
                                            , 'Folder 1'
                                            , 'Folder 2']
```

You can also set in your ~/.offlineimaprc the date from when to
start syncing like:
```console
maxage = 2006-01-01
```

Another tweak to make the initial sync faster is to modify the
maximum number of connections. Unfortunately, the server, in this
case the Exchange server, will close all the connections and fail.
So, even if it takes longer to download with one connection, you
will be able to download without any crashes.
```console
maxconnections = 1
```
Once your done with setting up the ~/.offlineimap, it's time to
setup the offlineimap systemd service and timer:
```console
 $ sudo cp ./mutt-setup/systemd/offlineimap.* /etc/systemd/system/.
```

Edit the /etc/systemd/system/offlineimap.service and add the path
to the offlineimap git repo.

```console
 $ sudo systemctl enable offlineimap.*
 $ sudo systemctl start offlineimap.*
```

<a name="msmtp"></a>
## 4. msmtp

Since the msmtp provided by the distro works just fine and doesn't
need any kind of fixup to provide the functionality we need, you
can install it from the distro repository:

```console
 $ sudo apt install msmtp
```

Copy the .msmtprc to home folder and make it r/w for the user only.

```console
 $ cp ./mutt-setup/dotfiles/.msmtprc ~/.
 $ chmod 600 ~/.msmtprc
```

Edit the ~/.msmtprc and add your email for both 'from' and 'user' entries
and the password.

<a name="mailcap"></a>
## 5. mailcap


