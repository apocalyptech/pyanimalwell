Python-Based CLI Animal Well Savegame Editor
============================================

`pyanimalwell` is a save editor (and data backend) for editing
[Animal Well](https://store.steampowered.com/app/813230/ANIMAL_WELL/) savegames.
It supports editing nearly everything stored in the savegames.  In addition
to the editing features you'd expect, it's got some features you might not
expect, including:

 - Importing/Exporting specific slots
 - Setting the Bunny Mural to its default, solved, or cleared state
 - Forcing Kangaroo spawns to a specific room
 - Respawning consumables (fruit, firecrackers) on the map
 - Clearing ghosts / Lighting Candles
 - Clearing out "illegal" pink-button presses (acquired via cheating) to
   avoid future savefile corruption
 - Image import/export to/from the "pencil" map layer

Table of Contents
-----------------

 - [Overview](#overview)
 - [Running / Installation](#running--installation)
 - [TODO](#todo)
 - [Usage](#usage)
   - [Showing Save Info](#showing-save-info)
   - [Fix Checksum](#fix-checksum)
   - [Choose Slot](#choose-slot)
   - [Import/Export Slots](#importexport-slots)
   - [Frame Seed](#frame-seed)
   - [Global Unlocks (Figurines, etc)](#global-unlocks-figurines-etc)
   - [Health](#health)
   - [Spawnpoint](#spawnpoint)
   - [Steps](#steps)
   - [Deaths](#deaths)
   - [Bubbles Popped](#bubbles-popped)
   - [Berries Eaten While Full](#berries-eaten-while-full)
   - [Game Ticks (Elapsed Time)](#game-ticks-elapsed-time)
   - [Wings (Flight)](#wings-flight)
   - [Boo](#boo)
 - [Library](#library)
 - [License](#license)

Overview
--------

The utility seems quite safe to use -- I've not ever experienced any problems
with it.  But make sure to bakup your saves before using this!

Work on decoding the savegame structure has mostly been done by
[Kein](https://github.com/Kein/), [just-ero](https://github.com/just-ero),
lipsum, and myself.  My own contributions were mostly at the beginning of
the process; Kein, just-ero, and lipsum have been responsible for the
majority of the save format at this point.  Many thanks to them for filling
things out!

A complete mapping of the savegame data can be found at
[Kein's awsgtools repo](https://github.com/Kein/awsgtools).  At time of
writing the primary format there is an
[010 Editor](https://www.sweetscape.com/010editor/) binary template
plus an [ImHex](https://imhex.werwolv.net/) pattern.  Other translations
may become available over time.  A human-readable document describing the
save format can be found [at the wiki of that repo](https://github.com/Kein/awsgtools/wiki/Format-Description)
as well, though at time of writing it's lagging behind the binary
template by quite a bit.

Running / Installation
----------------------

At the moment there is a temporary script right in the main project dir which
can be launched as so:

    ./aw.py --help

Or you can call the cli module directly, if you prefer:

    python -m animalwell.cli --help

TODO
----

The editor currently does not attempt to support *everything* inside the
savegames.  Some notable bits of data which can't be edited directly:

 - Crank status
 - Elevator status
 - Stalactite/Stalagmite/Icicle destruction
 - Some seemingly-unimportant flags have been omitted from a few areas
 - Achievements *(unsure if setting these in the save would actually make
   them activate on Steam)*
 - Game Options

While I don't have any current plans to support the above, there are a
few other things which would be nice eventually:

 - Mapping chests to their unlocks, so unlocking eggs (for instance) would
   mark the relevant chests as opened.  Vice-versa for disabling unlocks;
   it'd be nice to close the associated chest so it could be re-acquired.
   At the moment, chest opening/closing is all-or-nothing.
 - Similarly, mapping buttons/reservoirs to which doors they open would be
   nice, to be able to couple those a bit more closely.  At the moment,
   button/door states are all-or-nothing.

Usage
-----

*(this is still being filled in)*

Here is a detailed list of all the arguments available on the CLI editor.
Note that these arguments can be chained together pretty much as long as
you want.  For instance:

    aw.py AnimalWell.sav -i -s 0 --equip-enable all --inventory-enable pack --firecrackers 6 --upgrade-wand

### Showing Save Info

To show information about the save, including for any chosen slots, use
`-i`/`--info`:

    aw.py AnimalWell.sav -i

### Fix Checksum

The `--fix-checksum` option can be used to fix the savegame's checksum without
changing anything else.  (The utility will automatically update the checksum if
any other changes are made to the file.)  If the save has an invalid checksum,
Animal Well will spawn a Manticore friend to follow you around, so this can be
used to fix it in case any manual hex editing has been going on.

    aw.py AnimalWell.sav --fix-checksum

### Choose Slot

Most options in the editor operate on slot data, so this argument is generally
necessary.  `-s` or `--slot` can be used for this, to choose
a specific slot (`1`, `2`, or `3`), or to operate on *all* slots by
choosing `0`.

    aw.py AnimalWell.sav -i -s 1

Note that a few arguments (namely `--import-slot` and `--export-slot`)
do not allow using `0` to specify "all slots."

### Import/Export Slots

Slot data can be imported or exported to allow transferring slots without
also transferring "global" data (like collected figurines, or game options).
For example, to export slot 2 into the file `slot2.dat` with the
`--export`/`--export-slot` option:

    aw.py AnimalWell.sav -s 2 --export slot2.dat

Then that file could be imported into another slot with the `--import`/`--import-slot`
option:

    aw.py AnimalWell.sav -s 3 --import slot2.dat

Note that the only data checking the import process does is to ensure that
the file being imported is exactly the right filesize (149,760 bytes).

If the filename you attempt to export to already exists, this app will
prompt you if you want to overwrite.  To force overwriting without any
interactive prompt, use the `-f`/`--force` option:

    aw.py AnimalWell.sav -s 2 --export slot2.dat -f

### Frame Seed

The "frame seed" is presumably used for various randomizations in the
game, but the most noticeable effect is to determine which Bunny Mural
segment is presented to the user.  The mural segment shown is the frame
seed modulo 50, so you could theoretically get all of the segments
yourself by editing this value 50 times.  `--frame-seed` can be used
to alter the value:

    aw.py AnimalWell.sav --frame-seed 42

### Global Unlocks (Figurines, etc)

Some items unlocked in the game apply to all slots, such as the stopwatch,
pedometer, pink phones, and the various figurines which show up in the House.
The `--globals-enable` and `--globals-disable` arguments can be used to
toggle the various states.  The special value `all` can be used to toggle
all of them at once, and the argument can be specified multiple times.  See
the `--help` output for exactly which values can be used here.

    aw.py AnimalWell.sav --globals-disable stopwatch --globals-disable pedometer
    aw.py AnimalWell.sav --globals-enable all

### Health

Player health can be set with the `--health` option.  Note that this is the
total number of hearts, which includes your base health, any gold hearts,
and any "extra" blue hearts.  The game's maximum is probably 12, though I
haven't actually tested beyond that.

    aw.py AnimalWell.sav -s 1 --health 8

Relatedly, gold hearts can be added with the `--gold-hearts` option:

    aw.py AnimalWell.sav -s 1 --health 12 --gold-hearts 4

### Spawnpoint

The player's spawnpoint is stored in the game by room coordinates, and can
be set with the `--spawn` argument.  The upper-left room is at coordinate
2,4.  If there is a phone in the room that has been specified, the player
will spawn near the phone.  Otherwise, the player spawns in the upper-left-most
available spot in the room.

    aw.py AnimalWell.sav -s 1 --spawn 11,11

### Steps

You can set the number of steps the player has walked with the `--steps`
argument:

    aw.py AnimalWell.sav -s 1 --steps 42

The ingame pedometer rolls over to 0 after hitting 99,999.

### Deaths

You can set the number of recorded deaths using the `--deaths` option:

    aw.py AnimalWell.sav -s 1 --deaths 0

### Saves

You can set the number of saves that the player has performed using the
`--saves` option.  Note that the minimum legal value here is 1, since
the slot data will not be populated on-disk until the first savegame.

    aw.py AnimalWell.sav -s 1 --saves 1

### Bubbles Popped

The number of your B.Wand bubbles which have been popped by hummingbirds
can be altered with the `--bubbles-popped` argument:

    aw.py AnimalWell.sav -s 1 --bubbles-popped 42

### Berries Eaten While Full

The number of berries you've eaten while at full health can be altered
with the `--berries-eaten-while-full` argument:

    aw.py AnimalWell.sav -s 1 --berries-eaten-while-full 10

### Game Ticks (Elapsed Time)

The game keeps track of how many "ticks" have elapsed since the beginning
of the game.  The game runs at 60fps, so this value is *not* actually in
milliseconds.  There are technically two fields for this: one value which
is *just* ingame time (which does not seem to be used anywhere), and
another which includes all the time spent in "pause" menus.  There are
two arguments related to ticks.  First, `--ticks-copy-ingame` can be
used to copy your "ingame" counter over to the "including paused" counter,
in case you feel that your speed effort is being unfairly held back by
pausing:

    aw.py AnimalWell.sav -s 1 --ticks-copy-ingame

Alternatively, you can just set the numeric value for both at the same
time using `--ticks`:

    aw.py AnimalWell.sav -s 1 --ticks 0

### Wings (Flight)

A very late-game unlock gives you the ability to sprout wings and fly
around very quickly, by double-jumping.  This can be enabled or disabled
using `--wings-enable` and `--wings-disable`:

    aw.py AnimalWell.sav -s 1 --wings-enable
    aw.py AnimalWell.sav -s 1 --wings-disable

Library
-------

The data backend should be easily usable on its own, for anyone wishing
to make programmatic changes, or write other frontends.  At the moment
the best docs are just browsing through the code.  I apologize for my
frequent PEP-8 violations, lack of typing hints, and idiosyncratic
coding habits, etc.

A quick example of the kinds of things that would be possible:

```py
from animalwell.savegame import Savegame, Equipment, Equipped

with Savegame('AnimalWell.sav') as save:
    slot = save.slots[0]
    print(f'Editing slot {slot.index+1} ({slot.timestamp})...')
    print('Current equipment unlocked:')
    for equip in sorted(slot.equipment.enabled):
        print(f' - {equip}')
    slot.num_steps.value = 15
    slot.equipment.disable(Equipment.DISC)
    slot.equipment.enable(Equipment.WHEEL)
    slot.selected_equipment.value = Equipped.WHEEL
    save.save()
```

See also `animalwell/cli.py`, which is probably the best place to look for
examples of interacting with the data.

The objects are set up to allow both defining consecutive fields within
the savefile (without specifying manual offsets), and also skipping around
using manual offsets.  When offsets are specified, they are computed relative
to the "parent" object -- so for instance, the absolute offsets used in
the `Slot` class are relative to the start of the slot.  The fields created
while looping through the file are all subclasses of the base `datafile.Data`
class, which will end up computing and storing the *absolute* offset
internally.  The save data is mirrored in an internal `io.BytesIO` object,
which is where all changes are written.  It will not actually write back out
to disk until a `save()` call has been sent.

The data objects (subclasses of `datafile.Data`) try to be easy to read, and
can be interpreted as strings or in format strings, etc.  Subclasses of
`NumData` can often be used just as if they are numbers, supporting various
numerical overloads.  Bitfield classes can generally be acted on like
arrays, at least in terms of looping through the options.  `NumBitfieldData`
keeps an `enabled` set for ease of checking which members are enabled.

*Setting* new data should often be done via the `value` property rather than
setting it directly, though -- note in the example above where the `num_steps`
property is being set that way.  `value` should also be used if you need an
actual number, such as if using it in a list index or the like.

`NumChoiceData` objects are used where the value is expected to be one of
a member of an enum.  In these objects, `value` will always be the raw numeric
value still, but there's also a `choice` attribute which will correspond to
the proper enum item, if possible.  The class technically supports setting
values outside the known enum values, in which case `choice` will end up
being `None`.  Keep in mind that setting a value for these should still be done
via `value` rather than `choice`.

Apart from that, as I say, just looking through `cli.py` or the objects
themselves would probably be the best way to know how to use 'em.

License
-------

`pyanimalwell` is licensed under the [GNU General Public License v3](https://www.gnu.org/licenses/gpl-3.0.en.html).
A copy can be found at [LICENSE.txt](LICENSE.txt).

