# PHPloy
**Version 3.5.0**

PHPloy is a incremental Git FTP and SFTP deployment tool. By keeping track of the state of the remote server(s) it deploys only the files that were committed since the last deployment. PHPloy supports submodules, sub-submodules, deploying to multiple servers and rollbacks.

## Requirements

* PHP 5.3+ command line interpreter (CLI)
* SSH2 PECL extension (if using SFTP)
   * [Installation](http://php.net/manual/en/ssh2.installation.php)

Windows users can optionally [download AnsiCon](https://github.com/adoxa/ansicon/releases) to enable the display of colors in the command prompt.  Install it by running `ansicon -i` from a command prompt or "Run" window.

## Usage 

As any script, you can use PHPloy globally, from your `bin` directory or locally, from your project directory:


### Using PHPloy locally (per project)

1. Drop `phploy.phar` into your project.
2. Create the `deploy.ini` file.
3. Run `php phploy.phar` in terminal.


### Using PHPloy globally in Linux

1. Drop `phploy.phar` into `/usr/local/bin` and make it executable by running `sudo chmod +x phploy`.
2. Create the `deploy.ini` file inside your project folder.
3. Run `phploy` in terminal.

OR

You can add a symlink/symbolic link to `phploy.phar` in your `/usr/local/bin`, that way you can still update PHPloy, and you won't have to copy/paste new files everytime.

You can create the symlink very easy with this command:
`sudo ln -s ~/PHPloy/bin/phploy.phar /usr/local/bin/phploy`
In this case, I've placed the PHPloy folder in my home directory, you can just change the path to wherever PHPloy is placed.

Then you can run `phploy` in terminal. 


### Installing PHPloy globally in Windows

1. Extract or clone the PHPloy files into a folder of your choice
2. Ensure phploy.bat can find the path to php.exe by either:
    * Adding the path to php.exe to your system path
    * Manually adding the path inside phploy.bat
3. Add the phploy folder to your system path
4. Run `phploy` from the command prompt (from your repository folder)

Adding folders to your *system path* means that you can execute an application from any folder, and not have to specify the full path to it.  To add folders to your system path:

1. Press WINDOWS + PAUSE to open Control Panel > System screen
2. Click "Advanced System Settings"
3. Click "Environment Variables"
4. Under "System variables" there should be a variable called "Path".  Select this and click "Edit".
5. Keep the existing paths there, add a semi-colon `;` at the end and then type the location of the appropriate folder.  Spaces are OK, and no quotes are required.
6. Click OK


## deploy.ini

The `deploy.ini` file hold your credentials and it must be in the root directory of your project. Use as many servers as you need and whichever configuration type you prefer.

```ini
; This is a sample deploy.ini file. You can specify as many
; servers as you need and use normal or quickmode configuration.
;
; NOTE: If a value in the .ini file contains any non-alphanumeric 
; characters it needs to be enclosed in double-quotes (").

[staging]
scheme = sftp
user = example
; When connecting via SFTP, you can opt for password-based authentication:
pass = password
; Or private key-based authentication:
pubkey  = /path/to/public/key
privkey = /path/to/private/key
; If the private key is encrypted, you must also provide the passphrase:
keypass = passphrase
host = staging-example.com
path = /path/to/installation
port = 22
passive = true
; You can specify a list of patterns of files to be uploaded.
; Only files that match at least one of the patterns will be uploaded to the server.
; If a list of include patterns is not present, all files are considered
; by default (as if include[] = '*' was specified).
include[] = 'public_html/*'
; Files that should be ignored and not uploaded to your server, but still tracked in your repository
; This takes precedence over include[]
skip[] = 'src/*.scss'
skip[] = '*.ini'
skip[] = 'public_html/ignored/*'

[production]
quickmode = ftp://example:password@production-example.com:21/path/to/installation
passive = true
; Files that should be ignored and not uploaded to your server, but still tracked in your repository
skip[] = 'libs/*'
skip[] = 'config/*'
skip[] = 'src/*.scss'
```

If your password is missing in the `deploy.ini` file, PHPloy will interactively ask you for your password.

The first time it's executed, PHPloy will assume that your deployment server is empty, and will upload ALL the files of your project.  If the remote server already has a copy of the files, you can specify which revision it is on using the `--sync` command (see below).


## Multiple servers

PHPloy allows you to configure multiple servers in the deploy file and deploy to any of them with ease. 

By default PHPloy will deploy to *ALL* specified servers.  Alternatively, if an entry named 'default' exists in your server configuration, PHPloy will default to that server configuration. To specify one single server, run:

    phploy -s servername

or:

    phploy --server servername
    
`servername` stands for the name you have given to the server in the `deploy.ini` configuration file.

If you have a 'default' server configured, you can specify to deploy to all configured servers by running:

    phploy --all

## Rollbacks

**Warning: the --rollback option does not currently update your submodules correctly.  Until this is fixed, we recommend that you checkout the revision that you would like to deploy, update your submodules, and *then* run phploy.**

PHPloy allows you to roll back to an earlier version when you need to. Rolling back is very easy. 

To roll back to the previous commit, you just run:

    phploy --rollback

To roll back to whatever commit you want, you run:

    phploy --rollback="commit-hash-goes-here"

When you run a rollback, the files in your working copy will revert **temporarily** to the version of the rollback you are deploying. When the deployment has finished, everything will go back as it was.

Note that there is not a short version of `--rollback`.


## Listing changed files

PHPloy allows you to see what files are going to be uploaded/deleted before you actually push them. Just run: 

    phploy -l

Or:

    phploy --list

## Upload other files

To upload all files, even the ones not tracked by git (e.g. the Composer vendor directory), run:

    phploy -o

Or:

    phploy --others

Please keep in mind that **all** files not excluded in your deploy.ini will be uploaded.


## Updating or "syncing" the remote revision

If you want to update the `.revision` file on the server to match your current local revision, run:

    phploy --sync

If you want to set it to a previous commit revision, just specify the revision like this:

    phploy --sync="your-revision-hash-here"

## Submodules

Submodules are supported, but are turned off by default since you don't expect them to change very often and you only update them once in a while. To run a deployment with submodule scanning, add the `--submodules` paramenter to the command:

    phploy --submodules
    
## Purging

In many cases, we need to purge the contents of a directory after a deployment. This can be achieved by specifing the directories in `deploy.ini` like this:

    ; relative to the deployment path
    purge[] = "cache/"
    ; absolute path
    purge[] = "/public_html/wp-content/themes/base/cache/"

## How it works

PHPloy stores a file called `.revision` on your server. This file contains the hash of the commit that you have deployed to that server. When you run phploy, it downloads that file and compares the commit reference in it with the commit you are trying to deploy to find out which files to upload.

PHPloy also stores a `.revision` file for each submodule in your repository.


## Contribute

If you've got any suggestions, questions, or anything else about PHPloy, [you should create an issue here](https://github.com/banago/PHPloy/issues). 


## Credits

The people that have brought PHPloy to you are:

* [Baki Goxhaj](https://twitter.com/banago) - lead developer
* [Bruno De Barros](https://twitter.com/terraduo) - initial inspiration
* [Fadion Dashi](https://twitter.com/jonidashi) - contributor
* [Simon East](https://twitter.com/SimoEast) - contributor, Windows support 
* [Mark Beech](https://github.com/JayBizzle) - contributor 
* [Guido Hendriks](https://twitter.com/GuidoHendriks) - contributor 

## Version history

Please check [release history](https://github.com/banago/PHPloy/releases) for details.

