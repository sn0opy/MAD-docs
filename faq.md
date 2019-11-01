# FAQ

## General

### How many Mon/Quests/Raids can i scan with one device?

Nobody can tell that. That depends on your area e.g the amount of spawnpoints/stops/gyms in that area and the distance between them. Start with a small area, increase it over time and see how much is possible.

Rule of thumb: while moving, the game requests data every 10 seconds. So it takes *at least* 10 seconds per location to scan for data. An area with 150 coordinates takes roughly 25 minutes to scan for single device.

### Is it planned to see gym defenders + trainers in gyms?

No. Definetly not. To avoid drama *and* privacy concerns the main MAD developers decided leave out this kind of information from gym data.


## MAD

### All my stop and gym names are "unknown"

PogoDroid can't fetch this kind of information automatically in init mode. The game simply doesn't submit this information without clicking stops or gyms. You can either run quest mode to get the stop names and / or use the [intelimport.sh](https://mad-docs.readthedocs.io/en/latest/extras/scripts/#intel-importer-intelimport-sh) script to import those names and pictures from Ingress.

### How can i check if MAD receives data?

If you see a green SUCCESS line with "Processing GMO" in it, then leave it alone - it's working!

### Can i use multiple area fences in one area?

Yes. Just add the second area to the same geofence file like this:

```
[area1]
51.123, 9.234
51.124, 9.235
[area2]
12.345, 1.234
54.321, 2.345
```

### Madmin has missing images

Check your `pogoasset` config in `configs/config.ini` and use a full path (starting with `/` on Linux) instead of a relative path.

### How can i regenerate a route?

To renerate a route just delete the calc file in the `files` directory and restart MAD. It will notice that one or more files are missing and will recalculate them.

### How can i remove Eventspawnpoints?

Nia sometimes activates more spawnpoints for Community Day or other events. You shoud delete them after the event has ended so MAD will not consider them in the routecalc or PrioQ.

```SQL
DELETE FROM trs_spawn WHERE first_detection like '2019-01-01%'; 
```
Fill in a Date which is shortly after the event started but keep the `%`.

If you want, you can backup those spawns and just temporary insert them for the events. (Google "mysqldump" if you want to know how)

### How can i rescan quests?

Delete the quests from the `trs_quest` table and restart MAD. 

```SQL
TRUNCATE TABLE trs_quest;
```

## Game

### My character is stuck in the ocean and it's not moving

If you dont have a red GPS error at the top it means that RGC is working but didn't get any commands from the MAD server.

- Check if your phone registered to the MAD server. The log line should look like this: `[INFO] Client ORIGINNAME registering`
- Check if that phone has something to do according to your mappings. Have a look at your MAD logs for that.

### PoGo (sometimes) says that my phone has an unsupported OS

Sometimes it may just be a hickup, try a reboot

- Check SafetyNet status via [https://play.google.com/store/apps/details?id=com.scottyab.safetynet.sample](https://play.google.com/store/apps/details?id=com.scottyab.safetynet.sample) - your phone has to pass this check.
- Go into MagiskManager and repackage it if you have not done so already: MagiskManager > Setting > Repackage MagiskManager
- Add PoGo to Magisk Hide: MagiskManager > Magisk Hide > Check PoGo

### I can see the red error (70) sometimes

That's nothing to worry about. It's the way Pogodroid can scan IV.

### Quest mode doesn't click anything on the screen

- Check if you are using a correct Magisk version. 19.1, 19.2 and sometimes 19.3 blocking RGC to click on the screen. 19.0 will work just fine.
- Check if you have a [navigation bar](https://material.io/design/platform-guidance/android-bars.html#android-navigation-bar) on your screen. If yes: disable it with this adb command: `adb shell settings put global policy_control immersive.full=com.nianticlabs.pokemongo`. It will then be hidden in the game.

### Should i be worried about the popups on the phone?

Popups from PoGo like "You are moving too fast" and "Dont drink and drive" doesn't matter except for quest scanning. But MAD will handle them.

### RGC or Pogodroid are crashing randomly

Disable battery optimisations in Android and enable the OOM override option in Pogodroid/RGC

### My workers aren't following my priority queue / route on the map

In fact, they do. The RouteManager removes and entry from the prioq and assigns it to a worker. While the prio route has already been updated on the map, the worker position is only set when the worker arrives at its destination. Depending on your mode and your settings it can take several seconds before the worker marker arrives at the location where once has been a prioq coordinate.

### sql_mode error, MySQL strict mode, mysql mode.

For MAD to function properly you will need to adjust your MySQL/MariaDB server `sql_mode`. There are few modes that breaks MAD and you will be asked to to disable those, however for maximum comfort and to avoid problems in future updates we suggest disabling everything, not only those reported. 

Set your `sql_mode` to `NO_ENGINE_SUBSTITUTION` or even to empty.

This tutorial will cover Ubuntu/Debian way with some steps to reproduce. Make sure to run those commands as **root** (or with sudo).

**Step 1**. Find your MySQL/MariaDB main config and check what directory should we use. Run those commands and see if they report anything.
```
grep includedir /etc/my.cnf
grep includedir /etc/mysql/my.cnf
```
If you got `No such file or directory` two times it's time to consult your distro/system manual where is yours MySQL/MariaDB config file. 
Expected output it something like that (**do not use those dirs from example, use your own!**):
```
$ grep includedir /etc/mysql/my.cnf
!includedir /etc/mysql/conf.d/
!includedir /etc/mysql/mariadb.conf.d/
```
or 
```
# grep includedir /etc/mysql/my.cnf
!includedir /etc/mysql/conf.d/
```
The part after `!includedir` is the interesting part - it's directory where we will create our custom settings file. It will vary from distro/version - so always check it. If you have more than one result (like first example) select one directory - for now I will use `/etc/mysql/conf.d`.

**Step 2**. Make sure that this directory exists.
```
mkdir /etc/mysql/conf.d
```
If you got ```mkdir: cannot create directory ‘/etc/mysql/conf.d’: File exists``` then nothing to worry about - directory already there, go to step 3. If you got ```Permission denied``` then make sure to run this command as **root** or with **sudo**.

**Step 3**. Create new file `MAD.cnf` in that directory
```
nano /etc/mysql/conf.d/MAD.cnf
```
**Step 4**. Copy-paste (right mouse click in PuTTy) below content into that file and save it (`CTRL-o`, `enter`, `CTRL-x`)
```
[mysqld]
sql_mode="NO_ENGINE_SUBSTITUTION"
```
If it complains about ```Permission denied``` then go back step 3 and make sure you run is as **root** or with **sudo**.

**Step 5**. Restart MySQL/MariaDB to apply new settings. Here are few commands - one should work. Work from top - if you see that MySQL/MariaDB server was restarted there is no need to issue rest of commands - just covering more ground. Run as **root** or with **sudo**.
```
service mariadb restart
service mysql restart
service mysqld restart
/etc/init.d/mysql restart
```

### What's the difference betwen these scanning modes?

MITM is short for "Man In The Middle". PogoDroid will inject into the running game process to read the data which is received from the game server.

**mon_mitm**

`mon_mitm` will scan for mon within a 70 meter radius. By default, no mon gets encountered and checked for IV, unless you define a list (`mon_ids_iv`) of IDs that should be encountered. The order of the IDs is the priority of them. So, for example, put Snorlax before Pidgey to make sure PogoDroid will scan Snorlax first). PogoDroid has a built-in limit of 2 encounter per location.

**iv_mitm**

This mode is relying on already scanned and active mon in your DB (via `mon_mitm` for example). It will jump directly to them to do an IV check. `iv_mitm` will build up a "first in first out" queue.

**pokestops**

You can use this mode for two things. Quest scanning or leveling. Both require the OCR requirements being installed (python-pip packages, tesseract and opencv), no visible navigation bar (see [Phone Setup](http://https://mad-docs.readthedocs.io/en/latest/installation/phonesetup.html#final-steps)) and Magisk prior version 19.1. Magisk 19.1 and above won't work!

Quest scanning will walk on a pre-calculated route to every stop and spin it. When the area is set to `coords` in the walker, MAD will check every other stop in the area (even those who are not on the route). Those stops will be processed after the first round. This process will repeat itself three times. MAD is able to determinate the exact mon encounter and item type when picking up the quest.

The level option is basically the quest mode but without constantly clearing out the quests in the queststack and MAD will check if that stop is unique for the worker. In case it's a stop that has been visited in the past, it will be skipped.

**raids_mitm**

This mode is used to scan every gym and raid in a 490 meter radius. No interaction with any ingame objects is needed. MAD will only scan the gym color, the current gym defender, free slots and the raid or egg if present. (More about that in the FAQ).

**idle**

The phone will stop the game and do nothing.