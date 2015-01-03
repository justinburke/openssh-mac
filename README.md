![https://jcs.org/images/notaweblog/2011-04-19_cocoa-ssh-askpass.png](https://jcs.org/images/notaweblog/2011-04-19_cocoa-ssh-askpass.png)

### About
This is a copy of Apple's
[OpenSSH 6.2p2](http://opensource.apple.com/source/OpenSSH/OpenSSH-189/)
that is bundled with OS X 10.10, plus a modification to add a
`RequireKeyConfirmation` option.  This no longer includes the `AddKeysToAgent`
option modification, as it is no longer needed.

With `RequireKeyConfirmation` set to `yes` in `~/.ssh/config`, any identities
added to `ssh-agent` will require confirmation before use.  Combined with the
included `cocoa-ssh-askpass` wrapper around CocoaDialog, a GUI dialog will be
presented when SSH tries to use an unlocked identity stored in the agent.  This
applies to SSH spawned from a terminal (directly or through things like `git`),
from a forwarded agent, and from any GUI program that uses it in the background
to setup tunnels like Sequel Pro.

More information about agent confirmation can be read at
[http://jcs.org/macssh](http://jcs.org/macssh).

### Building
Run `xcodebuild` from the top directory.

### Installing
`sudo xcodebuild install` will install it into `/tmp/OpenSSH.dst` as usual.
Overlay its `/usr` directory on to `/` with
`sudo rsync -av /tmp/OpenSSH.dst/usr/. /usr/.`.
Avoid directly installing into `/` by overriding `DSTROOT` because of some
scary recursive `chmod`s and `chown`s that the XCode build script does (from
Apple).

Download and install [CocoaDialog](http://mstratman.github.com/cocoadialog/) to
`/Applications/Utilities`.  The `cocoa-ssh-askpass` wrapper that is installed
as `/usr/libexec/ssh-askpass` will look for CocoaDialog at
`/Applications/Utilities/CocoaDialog.app/Contents/MacOS/CocoaDialog`.

### Usage
Run `ssh-add -D` to delete all identities from the agent.

Add `RequireKeyConfirmation yes` in `~/.ssh/config`.

At the next SSH connection requiring a private key, `ssh` will try to start up
an agent and add the key as it did before, with the usual secure input window
asking for the key passphrase.  Leave the "Remember password in my keychain"
option unchecked.

On the next SSH connection requiring access to the agent that now has your
passphrase, `/usr/libexec/ssh-askpass` will be invoked to prompt for
confirmation.  If you have setup CocoaDialog properly, you should see a GUI
prompt asking for confirmation.  Verify that clicking cancel denies access to
your agent.
