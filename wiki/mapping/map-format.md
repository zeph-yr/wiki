---
sidebar: auto
prev: ./intermediate-mapping.md
next: false
description: Take a deep dive into the format for a Beat Saber map. 
---
# Map Format
There's some cases where, available tools and map editors do not provide some functionality that you want.
You might also feel inspired to create a tool or map editor for the fun of it.

One problem, though. What *exactly* goes into a map file? This is important to understand when developing
new tools, scripts, and editors.

This page will help you understand the internal workings of a full Beat Saber map, and provide information
about what each property and value represents.

## Schemas
A *schema* is essentially the set of rules to a specific kind of file. They have a list of rules that you
*may* break, and rules that you *cannot* break.

Beat Saber contains its own schema; it defines what the format for a Beat Saber level should be, and is
what we are going to break down.

### BeatSaver Schema
However, members of the community have gone together and made their own schema, which re-enforces rules set by Beat Saber's
own schema, but also standardizes and sets *new* rules to follow.

This community-made schema is used by BeatSaver, which [you can find here](https://github.com/lolPants/beatmap-schemas).
When uploading your map to BeatSaver, it checks against these schema files to determine whether or not it is appropriate
for upload. A major change in this schema is the inclusion of `_customData` objects scattered throughout map files, which
is covered in this breakdown.

You can read through the BeatSaver Schema to get a deeper understanding of what goes into a Beat Saber map, as well as
ensure that your own program outputs proper maps that are ready to be uploaded to BeatSaver.

## `Info.dat`
`Info.dat` is the main file for a Beat Saber map. It describes basic metadata about your map, as well as point to other
files to use for difficulties, cover art, and audio.

:::warning
It is extremely important that you get this correct, otherwise the map will fail to load in Beat Saber.
:::

### Base Object
Here is the basic format for an `Info.dat` file. Notice how difficulty beatmap sets are ommitted;
they'll be explained later down this page.

```json
{
  "_version": "2.0.0",
  "_songName": "Example Song",
  "_songSubName": "",
  "_songAuthorName": "Song Artist",
  "_levelAuthorName": "You",
  "_beatsPerMinute": 120,
  "_shuffle": 0,
  "_shufflePeriod": 0.5,
  "_previewStartTime": 31.5,
  "_previewDuration": 7,
  "_songFilename": "song.ogg",
  "_coverImageFilename": "cover.jpg",
  "_environmentName": "BigMirrorEnvironment",
  "_allDirectionsEnvironmentName" : "GlassDesertEnvironment",
  "_songTimeOffset": 0,
  "_customData": {
    // Any custom data will go here.
    // If empty, this should be removed entirely.
  },
  "_difficultyBeatmapSets": [
    // Difficulty beatmap sets (and difficulty beatmaps) are explained later down the page.
    // Check the sidebar!
  ]
}
```

#### _version
This field describes the version of the map format we are using. Currently, Beat Saber's map info format is on version `2.0.0`.

#### _songName
This field describes the name of your song.

#### _songSubName
This field describes any additional titles that could go into your song. These can include the following:

* Additional artists (Such as featured artists)
* Any variation in production (Song remix, VIP, etc.)

#### _songAuthorName
This field describes the main artist, group, band, brand, etc. for the song.

#### _levelAuthorName
This field describes the person who created the map. That's you! Or, whoever makes a map using your tool or level editor.

#### _beatsPerMinute
This describes the Beats Per Minute (BPM) of your song. This is a floating point number, so decimal BPMs are supported.

#### _shuffle
This and [`_shufflePeriod`](#shuffleperiod) are uncommon in the community. If your song has "swing" in it, where some beats
in a measure are intentionally offset from the rest, you can correct potential timing issues in your map by utilizing
`_shuffle` and [`_shufflePeriod`](#shuffleperiod).

`_shuffle` indicates how far objects will move when they are determined to be on a swing beat. A positive value means
they will be shifted forward in time, and a negative value means they will be shifted back in time.

The total amount they will be offset by is described in [`_shufflePeriod`](#shuffleperiod),
since they both work together to produce that value.

#### _shufflePeriod
`_shufflePeriod` is used to determine *when* a swing beat will occur. More specifically, it is the time (in beats)
where a swing beat will occur.

But unfortunately, it's more complicated than this. Beat Saber alternates between a swing beat and a non swing beat using
this value. For example, let's assume you have a `_shufflePeriod` of `0.25`. This tells Beat Saber that, every `0.25`
beats, it will alternate between a swing beat and a non swing beat, and will apply an offset if it lands on a swing beat.

The offset value that will be applied to objects on a swing beat is approximately equal to `_shuffle * _shufflePeriod` beats.

To hopefully help better understand this, here is a table of beats, whether or not they are on a swing beat, and the
*actual* beat objects at those times will spawn in at. For this example, we will assume that [`_shuffle`](#shuffle)
is `0.2`, and `_shufflePeriod` is `0.25`.

| Beat from Map File | Is Swing Beat? | Resulting Beat |
| -    | -   | - |
| 0    | No  | 0 |
| 0.25 | Yes | 0.3 |
| 0.5  | No  | 0.5 |
| 0.75 | Yes | 0.8 |
| 1    | No  | 1 |
| 1.25 | Yes | 1.3 |
| 1.5  | No  | 1.5 |
| 1.75 | Yes | 1.8 |

#### _previewStartTime
This controls the start time (in seconds) for the in-game preview of your map.
This is a floating point number, so decimals are supported.

#### _previewDuration
This controls the duration (in seconds) of the in-game preview of your map.
This is a floating point number, so decimals are supported.

#### _songFilename
This is the local location to your map's audio file. The standard practice is to have every map file in the same directory,
so in most cases, this is just the name and extension for your audio file (For example, `song.ogg`).

#### _coverImageFilename
This is the local location to your map's cover image. Both `.jpg` and `.png` are supported image types. Similar to
[`_songFilename`](#songfilename), this is most often just the name and extension for the cover image (For example, `cover.jpg`).

#### _environmentName
This defines the internal ID for the environment that the map uses. To get a complete list of valid environments, see the
`Info.dat Name`s of each environment in the [Environment Previews section](./basic-lighting.md#environment-previews).

#### _allDirectionsEnvironmentName
This defines the internal ID for the environment that the map uses when playing in 360 Degree or 90 Degree levels. This
is a required field, even if the level does not include any 360 or 90 Degree difficulties. To get a complete list of valid
360 environments, see the `Info.dat Name`s of each environment in the [Environment Previews section](./basic-lighting.md#environment-previews).

#### _songTimeOffset
This is Beat Saber's method for tackling off-sync audio. This offsets the audio in game,
based off the value of `_songTimeOffset` in seconds.

:::warning
Hit sounds are also affected by the same offset. We recommend the mapper sync up their audio file *before* mapping, as
described in the [Basic Audio Setup guide](./basic-audio.md), to circumvent any need for `_songTimeOffset` and related alternatives.
:::

#### _customData
This is an optional field that contains data unrelated to the official Beat Saber level format.
If no custom data exists, this object should be removed entirely.

The exact specifics of what goes in `_customData` is entirely dependent on community-created content that needs them.
As such, we cannot list all `_customData` fields here. You will have to do your own searching throughout the Beat Saber
community to find map editors, tools, or mods that use this `_customData` object.

#### _difficultyBeatmapSets
This is an array of all [Difficulty Beatmap Sets](#difficulty-beatmap-sets) defined in the map.

### Difficulty Beatmap Sets
Difficulty Beatmap Sets are groups of difficulties, all under one characteristic.
These represent your Standard, No Arrow, One Saber, and other characteristics.

```json
{
  // ... Rest of Info.dat ...
  "_songTimeOffset": 0,
  "_difficultyBeatmapSets": [
    {
      "_beatmapCharacteristicName": "Standard",
      "_difficultyBeatmaps": [
        // Difficulty beatmap are explained later down the page.
        // Check the sidebar!
      ]
    }
  ]
}
```

#### _beatmapCharacteristicName
This is the name of the characteristic attached to this beatmap set.

Listed below is all commonly used characteristics. While they have little to no "rules" attached to them in Beat Saber,
they still have an intended purpose, and should be followed by both the map editor and the mapper creating maps.

Certain characteristics, which are marked in the list below, do not belong to the base game; rather, they are added by
external mods such as SongCore. These modded characteristics will only work if the user has installed mods that add them,
and will *not* appear on unmodded copies of Beat Saber and could cause the map to not load.

|Characteristic Name|Included in Base Game|Intended Purpose|
|:-----------------:|:-------------------:|----------------|
|`Standard`|✔️|Vanilla maps following standard mapping guidelines.|
|`NoArrows`|✔️|Restrict notes to Dot (any direction) notes.|
|`OneSaber`|✔️|Restrict notes to Right (Blue) notes, and disables the Left (Red) saber.
|`360Degree`|✔️|Enables rotation events, with no restriction on total rotation.|
|`90Degree`|✔️|Enables rotation events, but restricts total rotation to 45 degrees to the left and right.|
|`Lightshow`|❌|Place for maps that only contains lighting events.|
|`Lawless`|❌|Modded maps and modcharts can safely go here. No rules should apply.|

#### _difficultyBeatmaps
This is an array of [Difficulty Beatmaps](#difficulty-beatmaps) defined within this beatmap set.

### Difficulty Beatmaps
Difficulty Beatmaps are each "Difficulty" of a map. They contain information that changes from difficulty to difficulty,
such as [Note Jump Speed](#notejumpmovementspeed), and the location of the difficulty file.

```json
{
  // ... Rest of Info.dat ...
  "_songTimeOffset": 0,
  "_difficultyBeatmapSets": [
    {
      "_beatmapCharacteristicName": "Standard",
      "_difficultyBeatmaps": [
        {
          "_difficulty": "ExpertPlus",
          "_difficultyRank": 9,
          "_beatmapFilename": "StandardExpertPlus.dat",
          "_noteJumpMovementSpeed": 18,
          "_noteJumpStartBeatOffset": 0,
          "_customData": {
            // Any custom data will go here.
            // If empty, this should be removed entirely.
          }
        }
      ]
    }
  ]
}
```

#### _difficulty
This is the internal difficulty, read by Beat Saber.

Contrary to what you might think, this is *not* just a normal string, but rather an Enum. Here is a list of all valid difficulties:

* `Easy`
* `Normal`
* `Hard`
* `Expert`
* `ExpertPlus`

#### _difficultyRank
This is the sorting order in the song select screen in Beat Saber.

While, yes, this is an ordinary integer, the widely-used [BeatSaver Schema](#beatsaver-schema) makes this another Enum,
based off of the aforementioned [`_difficulty`](#difficulty) value:

|`_difficulty`|BeatSaver's Expected `_difficultyRank`|
|-------------------|:-------------------:|
|`Easy`|1|
|`Normal`|3|
|`Hard`|5|
|`Expert`|7|
|`ExpertPlus`|9|

#### _beatmapFilename
This is the local location to the difficulty file, which contains the difficulty's notes, obstacles, and lighting events.

Similar to the [`_songFilename`](#songfilename) and [`_coverImageFilename`](#coverimagefilename) from earlier, in most
cases this is just the name and extension (always `.dat`) to the map file.

When creating *new* difficulties, it is recommended that the name be the Characteristic name for this difficulty's parent
[Beatmap Set](#difficulty-beatmap-sets), followed by the [`_difficulty`](#difficulty) value. For example, this particular
difficulty should have it's difficulty file be named `StandardExpertPlus.dat`.

#### _noteJumpMovementSpeed
Note Jump Movement Speed (Shortened to "Note Jump Speed", or just "NJS") is the velocity of objects approaching the player,
in meters per second. Info on recommended NJS values can be found on the
[Intermediate Mapping Page](./intermediate-mapping.md#note-jump-speed-spawn-distance).
This can be a floating point number for precise velocity.

This is used, along with the defined BPM of the song, to calculate 2 very important values,
called Jump Duration and Jump Distance.

* Jump Duration is the amount of beats where objects can be active.
* Jump Distance is the total amount of distance that objects need to travel within that Jump Duration.

The Player rests in the exact middle of both of these values, so most mappers find
it more convenient to have Half Jump Distance and Half Jump Duration.

* Half Jump Distance is the distance from the Player that objects spawn. Some mappers refer to this as the "Spawn Point".
* Half Jump Duration is the amount of beats that is needed to reach the Player. It is also the amount of beats,
  forward in time, where objects spawn.

#### _noteJumpStartBeatOffset
This value acts as a direct offset to the Half Jump Duration, explained in [`_noteJumpMovementSpeed`](#notejumpmovementspeed),
which in turn affects the Jump Distance. This can be a floating point number to achieve a precise Jump Duration.

#### _customData
This is an optional field that contains data unrelated to the official Beat Saber level format.
If no custom data exists, this object should be removed entirely.

The exact specifics of what goes in `_customData` is entirely dependent on community-created content that needs them.
As such, we cannot list all `_customData` fields here. You will have to do your own searching throughout the Beat Saber
community to find map editors, tools, or mods that use this `_customData` object.

## Difficulty File (v3)
Each Difficulty Beatmap contains a corresponding file which defines the notes, obstacles, events, and other objects
for that particular difficulty.

This version was introduced in Beat Saber version 1.20.0.

### Base Object

```json
{
  "version":"3.0.0", // There must be no whitespace after :
  "bpmEvents": [],
  "rotationEvents": [],
  "colorNotes": [],
  "bombNotes": [],
  "obstacles": [],
  "sliders": [],
  "burstSliders": [],
  "waypoints": [],
  "basicBeatmapEvents": [],
  "colorBoostBeatmapEvents": [],
  "lightColorEventBoxGroups": [],
  "lightRotationEventBoxGroups": [],
  "basicEventTypesWithKeywords": {},
  "useNormalEventsAsCompatibleEvents": false
}
```

#### version
This field describes the version of the map format we are using.

#### bpmEvents
This is an array of [BPM Event](#bpm-events) objects for the map.

#### rotationEvents
This is an array of [Rotation Event](#rotation-events) objects for the map.

#### colorNotes
This is an array of [Color Note](#color-notes) objects for the map.

#### bombNotes
This is an array of [Bomb Note](#bomb-notes) objects for the map.

#### obstacles
This is an array of [Obstacle](#obstacles-2) objects for the map.

#### sliders
This is an array of [Slider](#sliders-2) objects for the map.

#### burstSliders
This is an array of [Burst Slider](#burst-sliders) objects for the map.

#### waypoints
::: tip NOTE
This is a stub section.
:::
This is used to control BTS TinyTAN figures. Some information can be found in this
[document](https://docs.google.com/spreadsheets/d/1spW7LS-RvenLQBVXJl9w_iOwqr9r_ozxYo3JUlXq9Lc).

#### basicBeatMapEvents
This is an array of [Basic Event](#basic-beatmap-events) objects for the map.

#### colorBoostBeatmapEvents
This is an array of [Boost Event](#color-boost-beatmap-events) objects for the map.

#### lightColorEventBoxGroups
This is an array of Light Color Group Event objects for the map.
::: tip NOTE
This is a stub section. Documentation is a work in progress.
:::

#### lightRotationEventBoxGroups
This is an array of Light Rotation Group Event objects for the map.
::: tip NOTE
This is a stub section. Documentation is a work in progress.
:::

#### basicEventTypesWithKeywords
::: tip NOTE
This is a stub section. Documentation is a work in progress.
:::
  
#### useNormalEventsAsCompatibleEvents
This is a boolean.
::: tip NOTE
This is a stub section. Documentation is a work in progress.
:::

### Bpm Events

```json
{
  "b": 10.0,
  "m": 128.0,
}
```

:::danger
As of Beat Saber `1.20.0`, Official BPM Changes are still not complete. An official mapper has advised against using this
event in its current state.
:::

#### b
The time, in beats, where this object reaches the player.

#### m
A float representing the new bpm. Any `b` in objects after this point will be adjusted to the new bpm.

### Rotation Events

```json
{
  "b": 10.0, // Beat
  "e": 0,    // Event type
  "r": 15.0, // Rotation
}
```

#### b
The time, in beats, where this object reaches the player.

#### e
|`e`|Result|
|:---:|-----|
|`0`|Early rotation. Rotates future objects, while also rotating objects at the same time.|
|`1`|Late rotation. Rotates future objects, but ignores rotating objects at the same time.|

#### r
A float which represents clockwise rotation (as viewed from above).

### Color Notes

```json
{
  "b": 10.0, // Beat
  "x": 1,
  "y": 0,
  "c": 0,    // Color
  "d": 1,    // Direction
  "a": 0     // Angle offset
}
```

#### b
The time, in beats, where this object reaches the player.

#### x
An integer number, from 0 to 3, which represents the column where this note is located.
The far left column is located at index 0, and increases to the far right column located at index 3.

#### y
An integer number, from 0 to 2, which represents the layer where this note is located.
The bottommost layer is located at layer 0, and inceases to the topmost layer located at index 2.

#### c
An integer which represents the color of the note.
|`c`|Result|
|:---:|-----|
|`0`|Red|
|`1`|Blue|

#### d
This indicates the cut direction for the note.

|`d`|Result|
|:-------------------:|-------------------|
|`0`|Up|
|`1`|Down|
|`2`|Left|
|`3`|Right|
|`4`|Up Left|
|`5`|Up Right|
|`6`|Down Left|
|`7`|Down Right|
|`8`|Any (Dot Note)|

#### a
An integer number which represents the additional counter-clockwise angle offset applied to the note's cut direction in degrees.
This has no effect on angles created due to snapping (e.g. dot stack, slanted windows).

### Bomb Notes

```json
{
  "b": 10.0, // Beat
  "x": 1,
  "y": 0
}
```

#### b
The time, in beats, where this object reaches the player.

#### x
An integer number, from 0 to 3, which represents the column where this note is located.
The far left column is located at index 0, and increases to the far right column located at index 3.

#### y
An integer number, from 0 to 2, which represents the layer where this note is located.
The bottommost layer is located at layer 0, and inceases to the topmost layer located at index 2.

### Obstacles

```json
{
  "b": 10.0, // Beat
  "x": 1,
  "y": 0,
  "d": 5,    // Duration
  "w": 1,    // Width
  "h": 3     // Height
}
```

#### b
The time, in beats, where this object reaches the player.

#### x
An integer number, from 0 to 3, which represents the column where this obstacle is located.
The far left column is located at index 0, and increases to the far right column located at index 3.

#### y
An integer number, from 0 to 2, which represents the layer where base of the obstacle is located.
The bottommost layer is located at layer 0, and inceases to the topmost layer located at index 2.
|`y`|Result|
|:-------------------:|-------------------|
|`0`|Grounded obstacle|
|`1`|Prone obstacle|
|`2`|Crouch obstacle|

#### d
The time, in beats, that the obstacle extends for (duration).
While `d` can go into negative numbers, be aware that this has some unintended effects.

#### w
An integer which which represents the width of the obstacle.
While `w` can go into negative numbers, be aware that this has some unintended effects.

#### h
An integer numbers from 1 to 5, which represents the height of the obstacle.
While `h` can go into negative numbers, be aware that this has some unintended effects.

### Sliders
These describe arcs. If the head or tail of an arc matches a note's time and position,
then the arc will connect with the note and alter how the note is scored.

```json
{
  "b": 10.0,  // Head Beat
  "c": 0,     // Color
  "x": 1,     // Head x
  "y": 0,     // Head y
  "d": 1,     // Head direction
  "mu": 1.0,  // Head multiplier
  "tb": 15.0, // Tail Beat
  "tx": 2,    // Tail x
  "ty": 2,    // Tail y
  "tc": 1,    // Tail direction
  "tmu": 1.0, // Tail Multiplier
  "m": 1,     // Mid-anchor mode
}
```

#### b
The time, in beats, where this head of this object reaches the player.

#### c
An integer which represents the color of the note.
|`c`|Result|
|:---:|-----|
|`0`|Red|
|`1`|Blue|

#### x
An integer number, from 0 to 3, which represents the column where the head of the arc is located.
The far left column is located at index 0, and increases to the far right column located at index 3.

#### y
An integer number, from 0 to 2, which represents the layer where the head of the arc is located.
The bottommost layer is located at layer 0, and inceases to the topmost layer located at index 2.

#### d
An integer number which represents the head direction of the arc.

|`d`|Result|
|:-------------------:|-------------------|
|`0`|Up|
|`1`|Down|
|`2`|Left|
|`3`|Right|
|`4`|Up Left|
|`5`|Up Right|
|`6`|Down Left|
|`7`|Down Right|
|`8`|Any (Dot Note)|

#### mu
A float which represents how far the arc goes from the head of the arc.
If head direction is a dot, this does nothing.

#### tb
The time, in beats, where this tail of this object reaches the player.

#### tx
An integer number, from 0 to 3, which represents the column where the tail of the arc is located.
The far left column is located at index 0, and increases to the far right column located at index 3.

#### ty
An integer number, from 0 to 2, which represents the layer where the tail of the arc is located.
The bottommost layer is located at layer 0, and inceases to the topmost layer located at index 2.

#### tc
An integer number which represents the tail direction of the arc.

|`tc`|Result|
|:-------------------:|-------------------|
|`0`|Up|
|`1`|Down|
|`2`|Left|
|`3`|Right|
|`4`|Up Left|
|`5`|Up Right|
|`6`|Down Left|
|`7`|Down Right|
|`8`|Any (Dot Note)|

#### tmu
A float which represents how far the arc goes from the tail of the arc.
If tail direction is a dot, this does nothing.

#### m
An integer number which represents how the arc curves from the head to the mid point of the arc under certain conditions:

* Head and tail `x` are equal; and
* Head and tail cut direction are equal **OR** their angle difference is 180

:::warning NOTE
Currently angle difference is NOT an absolute value. These means only half of the opposing direction pairs will
meet these conditions. These pairs are (head -> tail):

* Down -> Up
* Right -> Left
* DownRight -> UpLeft
* UpRight -> DownLeft
:::

|`m`|Result|
|:-------------------:|-------------------|
|`0`|Straight|
|`1`|Clockwise|
|`2`|Counter-Clockwise|

### Burst Sliders
These describe chain and links. If the head of a chain matches a note's time and position,
then the chain will connect with the note and alter how the note is scored.

```json
{
  "b": 10.0,  // Beat
  "x": 1,     // Head x
  "y": 0,     // Head y
  "c": 0,     // Color
  "d": 1,     // Head direction
  "tb": 15.0, // Tail Beat
  "tx": 2,    // Tail x
  "ty": 2,    // Tail y
  "sc": 3,    // Segment count
  "s": 0.5    // Squish factor
}
```

#### b
The time, in beats, where this head of this object reaches the player.

#### c
An integer which represents the color of the note.
|`c`|Result|
|:---:|-----|
|`0`|Red|
|`1`|Blue|

#### x
An integer number, from 0 to 3, which represents the column where the head of the arc is located.
The far left column is located at index 0, and increases to the far right column located at index 3.

#### y
An integer number, from 0 to 2, which represents the layer where the head of the arc is located.
The bottommost layer is located at layer 0, and inceases to the topmost layer located at index 2.

#### d
An integer number which represents the head direction of the arc.

|`d`|Result|
|:-------------------:|-------------------|
|`0`|Up|
|`1`|Down|
|`2`|Left|
|`3`|Right|
|`4`|Up Left|
|`5`|Up Right|
|`6`|Down Left|
|`7`|Down Right|
|`8`|~~Any (Dot Note)~~ Functions as down.|

#### tb
The time, in beats, where this tail of this object reaches the player.

#### tx
An integer number, from 0 to 3, which represents the column where the tail of the arc is located.
The far left column is located at index 0, and increases to the far right column located at index 3.

#### ty
An integer number, from 0 to 2, which represents the layer where the tail of the arc is located.
The bottommost layer is located at layer 0, and inceases to the topmost layer located at index 2.

#### sc
An integer number, greater than 0, which represents the number of segments in the burst slider.
The head counts as a segment.

#### s
A float which represents squish factor. This is the proportion of how much of the path from `(x,y)` to `(tx, ty)`
is used by the chain. This does not alter the shape of the path.
Values greater than 1 will extend the path beyond the specified end point.

:::danger
Do not set squish factor to 0. This will crash the game.
:::

### Basic BeatMap Events

```json
{
  "b": 10.0, // Equivalent to _time
  "et": 1,   // Equivalent to _type
  "i": 1,    // Equivalent to _value
  "f": 1.0   // Equivalent to _floatValue
}
```

See [Events](#events-2) in v2 for information of what these equivalent properties represent.

### Color Boost Beatmap Events

```json
{
  "b": 10.0,
  "o": true
}
```

#### b
The time, in beats, where this object reaches the player.

#### o
A boolean which determines whether boost lighting is on or off.  

## Difficulty File (v2)
Each Difficulty Beatmap contains a corresponding file which defines the notes, obstacles, events,
and other objects for that particular difficulty.

Version `2.6.0` was introduced in Beat Saber version 1.20.0.  
Version `2.5.0` was introduced in Beat Saber version 1.18.0.  
Version `2.2.0` was introduced in Beat Saber version 1.13.1.

### Base Object

```json
{
  "_version": "2.6.0",
  "_notes": [],
  "_sliders": [], // Introduced in version 2.6.0
  "_obstacles": [],
  "_events": [],
  "_waypoints": [], // Introduced in version 2.2.0
  "_customData": {
    // Any custom data will go here.
    // If empty, this should be removed entirely.
  }
}
```

#### _version
This field describes the version of the map format we are using.

#### _notes
This is an array of [Note](#notes-2) objects for the map.

#### _sliders
This is an array of [Slider](#sliders-4) objects for the map.

#### _obstacles
This is an array of [Obstacle](#obstacles-4) objects for the map.

#### _events
This is an array of [Event](#events-2) objects for the map.

#### _waypoints
::: tip NOTE
This is a stub section.
:::
This is used to control BTS TinyTAN figures. Some information can be found in this
[document](https://docs.google.com/spreadsheets/d/1spW7LS-RvenLQBVXJl9w_iOwqr9r_ozxYo3JUlXq9Lc).

#### _customData
This is an optional field that contains data unrelated to the official Beat Saber level format.
If no custom data exists, this object should be removed entirely.

The exact specifics of what goes in `_customData` is entirely dependent on community-created content that needs them.
As such, we cannot list all `_customData` fields here. You will have to do your own searching throughout the Beat Saber
community to find map editors, tools, or mods that use this `_customData` object.

### Notes

```json
{
  "_time": 10,
  "_lineIndex": 1,
  "_lineLayer": 0,
  "_type": 0,
  "_cutDirection": 1,
  "_customData": {
    // Any custom data will go here.
    // If empty, this should be removed entirely.
  }
}
```

#### _time
The time, in beats, where this object reaches the player.

#### _lineIndex
An integer number, from 0 to 3, which represents the column where this note is located.
The far left column is located at index 0, and increases to the far right column located at index 3.

#### _lineLayer
An integer number, from 0 to 2, which represents the layer where this note is located.
The bottommost layer is located at layer 0, and inceases to the topmost layer located at index 2.

#### _type
This indicates the type of note there is. Currently, there are 4 known types, but 1 remains unused:

|`_type`|Result|
|:-------------------:|-------------------|
|`0`|Left (Red) Note|
|`1`|Right (Blue) Note|
|`2`|Unused|
|`3`|Bomb|

#### _cutDirection
This indicates the cut direction for the note.

|`_cutDirection`|Result|
|:-------------------:|-------------------|
|`0`|Up|
|`1`|Down|
|`2`|Left|
|`3`|Right|
|`4`|Up Left|
|`5`|Up Right|
|`6`|Down Left|
|`7`|Down Right|
|`8`|Any (Dot Note)|

#### _customData
This is an optional field that contains data unrelated to the official Beat Saber level format.
If no custom data exists, this object should be removed entirely.

The exact specifics of what goes in `_customData` is entirely dependent on community-created content that needs them.
As such, we cannot list all `_customData` fields here. You will have to do your own searching throughout the Beat Saber
community to find map editors, tools, or mods that use this `_customData` object.

### Sliders
These describe arcs. If the head or tail of an arc matches a note's time and position, then the arc will connect with
the note and alter how the note is scored.

```json
{
  "_colorType": 1,
  "_headTime": 10.0,
  "_headLineIndex": 0,
  "_headLineLayer": 0,
  "_headControlPointLengthMultiplier": 1.0,
  "_headCutDirection": 0,
  "_tailTime": 20.0,
  "_tailLineIndex": 1,
  "_tailLineLayer": 1,
  "_tailControlPointLengthMultiplier": 1.0,
  "_tailCutDirection": 1,
  "_sliderMidAnchorMode": 0,
  "_customData": {
    // Any custom data will go here.
    // If empty, this should be removed entirely.
  }
}
```

#### colorType
This indicates the color of the arc:

|`_colorType`|Result|
|:-------------------:|-------------------|
|`0`|Left (Red) Note|
|`1`|Right (Blue) Note|

#### headTime
The time, in beats, where this head of this object reaches the player.

#### headLineIndex
An integer number, from 0 to 3, which represents the column where the head of the arc is located.
The far left column is located at index 0, and increases to the far right column located at index 3.

#### headLineLayer
An integer number, from 0 to 2, which represents the layer where the head of the arc is located.
The bottommost layer is located at layer 0, and inceases to the topmost layer located at index 2.

#### headControlPointLengthMultiplier
A float which represents how far the arc goes from the head of the arc.
If head direction is a dot, this does nothing.

#### headCutDirection
An integer number which represents the head direction of the arc.

|`_headCutDirection`|Result|
|:-------------------:|-------------------|
|`0`|Up|
|`1`|Down|
|`2`|Left|
|`3`|Right|
|`4`|Up Left|
|`5`|Up Right|
|`6`|Down Left|
|`7`|Down Right|
|`8`|Any (Dot Note)|

#### tailTime
The time, in beats, where this tail of this object reaches the player.

#### tailLineIndex
An integer number, from 0 to 3, which represents the column where the tail of the arc is located.
The far left column is located at index 0, and increases to the far right column located at index 3.

#### tailLineLayer
An integer number, from 0 to 2, which represents the layer where the tail of the arc is located.
The bottommost layer is located at layer 0, and inceases to the topmost layer located at index 2.

#### tailControlPointLengthMultiplier
A float which represents how far the arc goes from the tail of the arc.
If tail direction is a dot, this does nothing.

#### tailCutDirection
An integer number which represents the tail direction of the arc.

|`_tailCutDirection`|Result|
|:-------------------:|-------------------|
|`0`|Up|
|`1`|Down|
|`2`|Left|
|`3`|Right|
|`4`|Up Left|
|`5`|Up Right|
|`6`|Down Left|
|`7`|Down Right|
|`8`|Any (Dot Note)|

#### sliderMidAnchorMode
An integer number which represents how the arc curves from the head to the mid point of the arc under certain conditions:

* Head and tail `LineIndex` are equal; and
* Head and tail `CutDirection` are equal **OR** their angle difference is 180

:::warning NOTE
Currently angle difference is NOT an absolute value. These means only half of the opposing direction pairs will meet
these conditions. These pairs are (head -> tail):

* Down -> Up
* Right -> Left
* DownRight -> UpLeft
* UpRight -> DownLeft
:::

|`_sliderMidAnchorMode`|Result|
|:-------------------:|-------------------|
|`0`|Straight|
|`1`|Clockwise|
|`2`|Counter-Clockwise|

#### customData
This is an optional field that contains data unrelated to the official Beat Saber level format.
If no custom data exists, this object should be removed entirely.

The exact specifics of what goes in `_customData` is entirely dependent on community-created content that needs them.
As such, we cannot list all `_customData` fields here. You will have to do your own searching throughout the Beat Saber
community to find map editors, tools, or mods that use this `_customData` object.

### Obstacles

```json
{
  "_time": 10,
  "_lineIndex": 1,
  "_type": 0,
  "_duration": 10,
  "_width": 1,
  "_customData": {
    // Any custom data will go here.
    // If empty, this should be removed entirely.
  }
}
```

#### _time
The time, in beats, where this object reaches the player.

#### _lineIndex
An integer number, from 0 to 3, which represents the column where the left side of the obstacle is located.
The far left column is located at index 0, and increases to the far right column located at index 3.

#### _type
An integer number which represents the state of the obstacle.

|`_type`|Result|
|:-------------------:|-------------------|
|`0`|Full height wall|
|`1`|Crouch/duck wall|

#### _duration
The time, in beats, that the obstacle extends for. While `_duration` can go into negative numbers,
be aware that this has some unintended effects.

#### _width
How many columns the obstacle takes up. A `_width` of `4` will mean that this wall will extend the entire playable grid.
While `_width` can go into negative numbers, be aware that this has some unintended effects.

#### _customData
This is an optional field that contains data unrelated to the official Beat Saber level format.
If no custom data exists, this object should be removed entirely.

The exact specifics of what goes in `_customData` is entirely dependent on community-created content that needs them.
As such, we cannot list all `_customData` fields here. You will have to do your own searching throughout the Beat Saber
community to find map editors, tools, or mods that use this `_customData` object.

### Events

```json
{
  "_time": 10,
  "_type": 1,
  "_value": 3,
  "_floatValue" : 1.00, // Introduced in version 2.5.0
  "_customData": {
    // Any custom data will go here.
    // If empty, this should be removed entirely.
  }
}
```

#### _time
The time, in beats, where this object reaches the player.

#### _type
An integer number which represents what exact kind of event this object represents.

|`_type`|Result|
|:-------------------:|-------------------|
|`0`|Controls lights in the `Back Lasers` group.|
|`1`|Controls lights in the `Ring Lights` group.|
|`2`|Controls lights in the `Left Rotating Lasers` group.|
|`3`|Controls lights in the `Right Rotating Lasers` group.|
|`4`|Controls lights in the `Center Lights` group.|
|`5`|(Previously unused) Controls boost light colors (secondary colors).|
|`6`|(Previously unused) Controls extra left side lights in some environments.|
|`7`|(Previously unused) Controls extra right side lights in some environments.|
|`8`|Creates one ring spin in the environment.|
|`9`|Controls zoom for applicable rings. Is not affected by [`_value`](#value).|
|`10`|(Previously unused) (Previously Official BPM Changes.)<br/>Billie environment - Controls left side lasers|
|`11`|(Previously unused) <br/>Billie environment - Controls right side lasers.|
|`12`|Controls rotation speed for applicable lights in `Left Rotating Lasers`.|
|`13`|Controls rotation speed for applicable lights in `Right Rotating Lasers`.|
|`14`|(Previously unused) 360/90 Early rotation. Rotates future objects, while also rotating objects at the same time.|
|`15`|(Previously unused) 360/90 Late rotation. Rotates future objects, but ignores rotating objects at the same time.|
|`16`|Interscope environment - Lowers car hydraulics<br/>Gaga environment - Controls middle left tower height|
|`17`|Interscope environment - Raises car hydraulics<br/>Gaga environment - Controls middle right tower height|
|`18`|Gaga environment - Controls outer left tower height|
|`19`|Gaga environment - Controls outer right tower height|

:::danger
Just because an event type is listed as unused, does *not* mean you are freely available to use it!

Beat Games is known to repurpose previously unused event types for certain features, such as the introduction of 360&deg;
/ 90&deg; levels. This has broken some Beat Saber maps that make use of legacy MediocreMapper BPM Changes, as well as maps
that used Custom Platforms that took advantage of the unused event types.
:::

#### _value
Depending on the aforementioned [`_type`](#type) of the event, the `_value` of it can do different things.

##### Controlling Lights
It's default behavior is controlling brightness and color of lights, and follows this table:

|`_value`|Result|
|:-------------------:|-------------------|
|`0`|Turns the light group off.|
|`1`|Changes the lights to blue, and turns the lights on.|
|`2`|Changes the lights to blue, and flashes brightly before returning to normal.|
|`3`|Changes the lights to blue, and flashes brightly before fading to black.|
|`4`|(Previously Unused.)<br/>Changes the lights to blue by fading from the current state.|
|`5`|Changes the lights to red, and turns the lights on.|
|`6`|Changes the lights to red, and flashes brightly before returning to normal.|
|`7`|Changes the lights to red, and flashes brightly before fading to black.|
|`8`|Changes the lights to red by fading from the current state.|

`_value` 4 and 8 were introduced in Beat Saber version `1.18.0` (Billie Eilish patch). These events will only transition
 from Off and On (0, 1, and 4 )events. They will do nothing if transitions fade and flash events (2, 3, 6, and 7).

##### Controlling Boost Colors
|`_value`|Result|
|:-------------------:|-------------------|
|`0`|Turns the event off - switches to first (default) pair of colors.|
|`1`|Turns the event on - switches to second pair of colors.|

##### Controlling Rings
When the event is used to control ring zoom, the `_value` of the event does nothing.

When the event is used to control ring spin, the `_value` only affects cars in the Interscope environment and does
nothing in other environments.

##### Controlling Cars
|`value`|Result|
|:-------------------:|-------------------|
|`0`|Affects all the cars. Does not affect hydraulics.|
|`1`|Affects all the cars.|
|`2`|Affects the left cars.|
|`3`|Affects the right cars.|
|`4`|Affects the front-most cars.|
|`5`|Affects the front-middle cars.|
|`6`|Affects the back-middle cars.|
|`7`|Affects the back-most cars.|

##### Official BPM Changes (before version 2.5.0)
When the event is used to control the BPM, the `_value` represents the new BPM.

The new BPM does not shift internal [`_time`](#time-2) values for future objects. Instead, it essentially recalculates
internal game values (Such as Half Jump Duration and Jump Distance) to match the effect of playing the map at the new BPM.

One caveat to this is that the `_value` must *always* be an integer, and does not support floating point numbers (No decimals).

:::warning
As of Beat Saber `1.10.0`, Official BPM Changes are broken, and produce unwanted effects when used in a level.

If you absolutely want to work around this, you must create a new BPM Change event so that:

1. This new event *must* have the same exact [`_time`](#time-2) as the BPM Change event you want to trigger correctly.
2. This new event *must* have the same `_value` as the previous BPM Change,
  or the [`_beatsPerMinute`](#beatsperminute) defined in [`Info.dat`](#info-dat).
3. This new event *must* occur before the BPM Change you want to trigger correctly,
  *even if they share the same [`_time`](#time-2) values.*
:::

##### Controlling Laser Rotation Speed
When the event is used to control laser speed for a group of lights, the `_value`
is used as a multiplier to their base rotational velocity.

If `_value` is `0`, the random rotation offset for each laser will also be reset,
causing all rotating lasers to line up perfectly.

##### Controlling 360/90 Rotation
When the event is used to control rotation in a 360/90 degree level, the `_value`
is used to add rotation equal to the following table:

|`_value`|Result|
|:-------------------:|-------------------|
|`0`|60 Degrees Counterclockwise|
|`1`|45 Degrees Counterclockwise|
|`2`|30 Degrees Counterclockwise|
|`3`|15 Degrees Counterclockwise|
|`4`|15 Degrees Clockwise|
|`5`|30 Degrees Clockwise|
|`6`|45 Degrees Clockwise|
|`7`|60 Degrees Clockwise|

#### _floatValue
Depending on the aforementioned [`_type`](#type) of the event, the `_floatValue` of it can do different things.

##### Controlling Lights
When the event is used to control lights, the `_floatValue` determines the brightness of the light.

##### Official BPM Changes
:::danger
As of Beat Saber `1.18.0`, Official BPM Changes are still not complete. An official mapper has advised against using this
event in its current state.
:::
When the event is used to control the BPM, the `_floatValue` represents the new BPM. This will also alter the Note Jump Speed
proportional to the change in BPM.

#### _customData
This is an optional field that contains data unrelated to the official Beat Saber level format.
If no custom data exists, this object should be removed entirely.

The exact specifics of what goes in `_customData` is entirely dependent on community-created content that needs them.
As such, we cannot list all `_customData` fields here. You will have to do your own searching throughout the Beat Saber
community to find map editors, tools, or mods that use this `_customData` object.

## Credits
The content on this page was authored by [Caeden117](./mapping-credits.md#caeden117) and
[Bullet](./mapping-credits.md#bullet) with help from [Kival Evan](./mapping-credits.md#kival-evan)
and GalaxyMaster.
