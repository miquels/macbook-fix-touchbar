# macbook-fix-touchbar

Workaround for macbooks rebooting because the touchbar got hung.

On macbooks with a touch bar, sometimes the hardware of the touchbar
will get confused and fail to respond. Usually you'll also see the
touch bar flashing, or going completely black.

After 180 seconds the macos `watchdogd` will then conclude that there is
a fatal hardware problem, and crash/reboot the system.

There are 2 curious facts about this problem:
- it doesn't happen while you're typing / using the touchpad
- killing the `TouchBarServer` resets the hardware _and the watchdog_

So the `fix-touchbar` script runs in the background, and every few seconds
it checks:

- are we idle more than 60 seconds (not typing etc)
- in that case, restart the TouchBarServer process once a minute.

This seems to work. Most of the time, at least.


## Install:

For a script that runs as a normal user to be able to kill the (root) `TouchBarServer`
process, you need to add a line to the end of the _User Specification_ part of
the `/etc/sudoers` file. You can do this by running `visudo` and adding this:

```
ALL             ALL=(root) NOPASSWD: /usr/bin/pkill -x TouchBarServer
```

Note that this will enable any user on the macbook to issue the command
`sudo pkill -x TouchBarServer` - which should not really be a problem.

Then, run these commands to install the script to run as a daemon:

```
cp fix-touchbar ~/Library/Scripts
chmod 755 ~/Library/Scripts/fix-touchbar
cp nl.z42.FixTouchBar.plist ~/Library/LaunchAgents/
chmod 644 ~/Library/LaunchAgents/nl.z42.FixTouchBar.plist
launchctl enable gui/$UID/nl.z42.FixTouchBar
launchctl start nl.z42.FixTouchBar
```

Good luck!
