Freenet & mercurial

# Licence
This document and pyFreenetHg is published under the strong WTFPL [license](COPYING).

# About pyFreenetHg
Mercurial is written in python and has a plugin system. So it should not be to much difficulty to get it
working with freenet.

```
the basic idea: wrap around (selected) commands and use freenet as transport instead of files
     hg fcp-bundle   -> wraps around hg bundle and put a chk/usk instead a local file
     hg fcp-unbundle -> wraps around hg unbundle and get it from a chk/usk instead a local file	
     hg fcp-updatestatic -> do a commit hook upload, useful if the insert started from commit hook was failed
     
commit hooks:
     updatestatic_hook -> does a full upload on each commit
     updatestatic_hook2 -> upload is only triggered on keyword in commit message
     updatestatic_hook3 (in progress) -> upload is only triggered on keyword in commit message
                                      -> SiteToolPlugin is used to reduce upload size on large trees (if plugin is installed)     
```

# Requirements
python >= 2.5<br>
mercurial >= 1.1.2+<br>
last not least: a running freenet node ;)<br>
knowledge about hg native commands `bundle`/`unbundle`<br>
optional: <a href="/USK@MYLAnId-ZEyXhDGGbYOa1gOtkZZrFNTXjFl1dibLj9E,Xpu27DoAKKc8b0718E-ZteFrGqCYROe7XBBJI57pB4M,AQACAAE/SiteToolPlugin/6/" target="_blank">SiteToolPlugin</a>

# Installation
*Important hint:* Do not store url encoded keys into hgrc (`USK%40MY`). Python does not like `%` here!

Download [FreenetHg.py](FreenetHg.py) and put it somewhere. Add the following to your `~/.hgrc`:
```
[extensions]
freenethg=/path/to/FreenetHg.py
```

# Configuration
Most of the configuration is taking place on a per-repository basis. The config file for this can
be found in each mercurial repository in directory `.hg/hgrc`. Depending on how many different
repositories you manage, some of the config settings can also be placed in your global config
file `~/.hgrc`.

## Configuration wizard
pyFreenetHg provides a simple setup wizard that configure fcp settings and hooks.
```
hg setupwitz
```

## Default settings
If fcp settings are neither configured in hgrc nor environment variables are set nor given
as commandline parameters, the fcp settings defaults to following values:
* fcp host: 127.0.0.1
* fcp port: 9481
* fcp timeout: 30 minutes

## Environment variables
The following environment variables are supported:
* `FCP_HOST` - a string that represents the fcp host adress
* `FCP_PORT` - the fcp port number (1-65365)
* `FCP_TIMEOUT` - the connection timeout in seconds
* `FCP_NOVERSION` - if set the fcp version check is omitted

## Username
To prevent identity leaks you should add a username to each `./hg/hgrc`:
```
[ui]
username = Your Name
```
Examples:<BR>
username = Alice<BR>
username = bob@test.com<BR>
username = Charlie <ch@devs.freemail\><BR>

## Hooks
Usually, repositories in freenet can be updated after committing with `hg fcp-updatestatic`
(executed in your repository directory). To automate this, several hooks can be used.
Put one of these in per-repository `./hg/hgrc`:

### updatestatic_hook
This is the most simple hook. The repository gets updated on every commit.
```
[hooks]
commit = python:freenethg.updatestatic_hook

[freenethg]
inserturi = USK@foo,bar/Name/1
requesturi = USK@oof,rab/Name/1
fcphost = 127.0.0.1
fcpport = 9481
```

### updatestatic_hook2
With this hook configuration, update of repository only takes place if the given `uploadkeyword`
is in the commit message. (The repository can be updated at any time with `hg fcp-updatestatic`).
```
[hooks]
commit = python:freenethg.updatestatic_hook2

[freenethg]
inserturi = USK@foo,bar/Name/1
requesturi = USK@oof,rab/Name/1
fcphost = 127.0.0.1
fcpport = 9481
uploadkeyword = triggerword
```

### updatestatic_hook3
This hook is still under development. It will use the SiteToolPlugin to reduce the size of inserts for big repositories.
```
[hooks]
commit = python:freenethg.updatestatic_hook3

[freenethg]
inserturi = USK@foo,bar/Name/1
requesturi = USK@oof,rab/Name/1
fcphost = 127.0.0.1
fcpport = 9481
uploadkeyword = triggerword
```

# Usage
## Command line parameters
### fcp configuration parameters
<dl>
<dt>--fcphost hostnameorip</dt><dd>specify fcphost if not 127.0.0.1</dd>
<dt>--fcpport portnumber</dt><dd>specify fcpport if not 9481</dd>
<dt>--fcptimeout seconds</dt><dd>specify fcp timeout if not 30 minutes</dd>
<dt>--fcplog</dt><dd>log fcp to mercurials output</dd>
<dt>--fcpnoversion</dt><dd>omit fcp version check</dd>
</dl>

### fcp put parameters
<dl>
<dt>--fcpdontcompress</dt><dd>if specified node compression is turned off</dd>
<dt>--globalput</dt><dd>use gloabal queue and persistance "forever" for fcp put</dd>
</dl>

### notify parameters
<dl>
<dt>--nonotify</dt><dd>suppress all notifies</dd>
</dl>

## Sharing Changes
### One-shot Repository
To insert a repository with a unique key, change to your repository working directory and use this command:
```
hg fcp-updatestatic --uri=CHK@
```
This circumvents your `inserturi` settings of the config file and uses a simple CHK key.

### Creating/updating a more permanent Repository
To create and update a repository under a USK, just change to your repository working directory and use this command:
```
hg fcp-updatestatic
```
This will use the `inserturi` from your configfile. The node will take care of USK versioning.

### Creating bundles
You can bundle specific changesets and insert them as CHK. Example:
```
hg fcp-bundle --base 5345e333a19b --rev 80f52220ea7c
```
This creates a hg bundle of changes between revision `5345e333a19b` and `80f52220ea7c` and inserts this as CHK.
For more information on bundle command, see the mercurial documentation.

## Getting Changes
### Unbundling
To merge a bundled changeset into your repository, change to the working directory and use:
```
hg fcp-unbundle CHK@....
```
#### Without pyFreenetHg installed or version 0.1.1 or earlier
##### Fetching (cloning) a repository
To clone another repository, just use the normal hg command `clone` and use static-http as transport:
```
hg clone static-http://127.0.0.1:8888/USK@..../<version>
```
##### Pulling changes from a repository
The same applies for pulling changes into an existing repository:
```
hg pull static-http://127.0.0.1:8888/USK@..../<version>
```

#### This pyFreenetHg version or later installed
##### Fetching (cloning) a repository
To clone another repository, just use the normal hg command `clone` and use FCP (Freenet Client Protocol) as transport:
```
hg clone fcp://127.0.0.1:9481/USK@..../<version>
```
##### Pulling changes from a repository
The same applies for pulling changes into an existing repository:
```
hg pull fcp://127.0.0.1:9481/USK@..../<version>
```
##### FCP uri scheme
```
fcp://<user>:<password>@<host>:<port>/<freenetkey>;<connectionparams>?<commandparams>
```
The fcp uri scheme looks like the http uri scheme. Username and password are defined for future use, but currently ignored.
If the host or port is not given, the value from hgrc or environment is taken. If none is found, it defaults to 127.0.0.1:9481<br>
<ul>
<li>connection parameters. have only effect if new connections are created
<dl>
<dt>FCPLog</dt><dd>if the parameter is set, any value turns logging on (<em>default: off</em>)</dd>
<dt>NoVersion</dt><dd>if the parameter is set, any value turns the fcp version check off (<em>default: on</em>)</dd>
<dt>Timeout</dt><dd>set timeout to the amount of seconds (<em>default: 1800</em>)</dd>
</dl>
</li>
<li>command parameters
<dl>
<dt>Priority</dt><dd>the priority class for the request (0...6) (<em>default: 1</em>)</dd>
<dt>MaxRetries</dt><dd>set the max retry count. -1 retries forever (<em>default: 5</em>)</dd>
</dl>
</li>
</ul>
Sample uris:

```
fcp://127.0.0.1:9481/USK@fQGiK~CfI8zO4cuNyhPRLqYZ5TyGUme8lMiRnS9TCaU,E3S1MLoeeeEM45fDLdVV~n8PCr9pt6GMq0tuH4dRP7c,AQACAAE/freenethg/54/
fcp:///USK@fQGiK~CfI8zO4cuNyhPRLqYZ5TyGUme8lMiRnS9TCaU,E3S1MLoeeeEM45fDLdVV~n8PCr9pt6GMq0tuH4dRP7c,AQACAAE/freenethg/54/
fcp:USK@fQGiK~CfI8zO4cuNyhPRLqYZ5TyGUme8lMiRnS9TCaU,E3S1MLoeeeEM45fDLdVV~n8PCr9pt6GMq0tuH4dRP7c,AQACAAE/freenethg/54/ 
fcp://127.0.0.1:9481/USK@fQGiK~CfI8zO4cuNyhPRLqYZ5TyGUme8lMiRnS9TCaU,E3S1MLoeeeEM45fDLdVV~n8PCr9pt6GMq0tuH4dRP7c,AQACAAE/freenethg/54/;FCPLog=true&TimeOut=300?Priority=1&MaxRetries=5
fcp://127.0.0.1:9481/USK@fQGiK~CfI8zO4cuNyhPRLqYZ5TyGUme8lMiRnS9TCaU,E3S1MLoeeeEM45fDLdVV~n8PCr9pt6GMq0tuH4dRP7c,AQACAAE/freenethg/54/?Priority=0&MaxRetries=-1
```

# Templates
## Repository Indexpage Template
The standard hg repository is not a regular freesite, so you get usually a "Not in Archive" error
if you access the repository uri in fproxy.<br>
To resolve this problem, pyFreenetHg generates a simple index page.<br>

You can provide your own html template by adding this option to section `freenethg`:

```
[freenethg]
indextemplate = /path/to/template
```

You can use variables in the template which will be replaced by pyFreenetHg.<br>
At the moment following variables are available:
* $uri - the URI of your repository
* $fmsuser - your fms username, if provided in the config (see section about notifications)

## Bundle Indexpage Template
This is still work in progress. Bundles will get a history freesite with list of changeset for a specific repository.

# Notifications
pyFreenetHg can notify configurable groups of people about your repository and bundle inserts. At
the moment, only FMS is supported, but email and several other will follow soon.

If you do an insert via commandline notify can be suppressed with `--nonotify`.

First, put a `notify=<group>` in your `.hg/hgrc`, then add a special config section for this group.

```
[freenethg]
notify = name name2 ...

[notify_name]
type = fmsnntp
.
.
.

[notify_name2]
type = freemail
.
.
.

```

Look at the example for FMS notification for better explanation.

## FMS
This is an example notification config for FMS:

```
[freenethg]
.
.
notify = fmstest

[notify_fmstest]
type = fmsnntp
updatestatic_message_template = /path/to/fmstemplate.txt (optional)
bundle_message_template = /path/to/fmstemplate_bundle.txt (optional)
fmsuser = Alice
fmshost = 127.0.0.1
fmsport = 1119
fmsgroups = pyfreenethg, programming
```

This will send an FMS post to groups `pyfreenethg`, `programming` with FMS user `Alice` everytime
you execute one of the commands fcp-updatestatic or fcp-bundle.

### Custom message templates
You can specify custom message templates. The first line of the template will be used as addition
to the Subject line. The rest is treated as body.

Following variables are available for updatestatic templates:
* $uri - the URI of your repository
Following variables are available for bundle templates:
* $base - the base changeset of your bundle
* $rev - the revision of your bundle
Both of these variables are taken from the commandline of your `hg fcp-bundle` command, e.g.:
```
hg fcp-bundle --base ff3a3454cd13 --rev c293b1bd3182
```

## Other ways to get informed on repository updates
* Use -1 instead 1 as edition number to force the node to look for newer editions before delivering data
* Add the repository USK to your bookmarks, so you get notified if a new edition is found

<!-- h3>eclipse</h3>
<p>modified MercurialEclipse plugin (vectrace, 0.1.102) to fire fcp-createstatic and fcp-updatestatic from within eclipse</p>
<p>use this key as eclipse update site or open it in your browser and download &amp; install it manually.</p>
<p><a href="USK@MYLAnId-ZEyXhDGGbYOa1gOtkZZrFNTXjFl1dibLj9E,Xpu27DoAKKc8b0718E-ZteFrGqCYROe7XBBJI57pB4M,AQACAAE/FreenetUpdateSite/1/" target="_blank">http://127.0.0.1:8888/USK@MYLAnId-ZEyXhDGGbYOa1gOtkZZrFNTXjFl1dibLj9E,Xpu27DoAKKc8b0718E-ZteFrGqCYROe7XBBJI57pB4M,AQACAAE/EclipseUpdateSite/1/</a></p -->
