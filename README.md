# Pazel

:chart_with_upwards_trend: Python tool to plot azimuth and elevation of astronomical objects

## About Pazel

Pazel (/pǽzl/) is a single python script that plots azimuth (Az) and elevation (El) of astronomical objects for the purpose of planning an observation or monitoring objects during observation.
Pazel uses Google Maps API to automatically retrieve latitude, longitude, and timezone of an observation location from free format words.
It also uses PyEphem to compute Az and El of objects such as the sun, planets, and user defined ones under the given location and date.
After computing Az and El of objects, Pazel provides three way to plot: show and save mode for a day plot of them; list mode for monitoring them in terminal in real time.

## Dependencies
Pazel requires the following libraries in addition to Python standard ones.
All of them can be easily installed using pip.

+ pyephem: calculating Az and El of objects
+ numpy: calculating other parts
+ matplotlib: plotting figures
+ seaborn: making figures more beautiful (optional)

## Tutorial

To use Pazel in any modes, you must execute pazel (`$ ./pazel`) with several auguments.
By default, the sun and planets (described in solar_objects.json) are computed and plotted.
For more information about the arguments, type `./pazel -h` to show a help.
Note that Pazel requires an internet connection *for the first time* you spacify a new location.

### Show/save mode

These modes use matplotlib to show or save a day plot of Az and El of objects.
To show Az and El of solar objects seen from Mitaka, Tokyo on 1st Jan 2016, for example:

```bash
$ ./pazel -m show -l mitaka tokyo -d 2016-01-01
```

To show it with the x axis of LST (Local Sidereal Time):

```bash
$ ./pazel -m show -l mitaka tokyo -d 2016-01-01 --tz LST
```

To save it, replace `show` in the arguments with `save`:

```bash
$ ./pazel -m save -l mitaka tokyo -d 2016-01-01 --tz LST
```

To use user defined objects (described in users_objects.json, for example):

```bash
$ ./pazel -m show -l mitaka tokyo -d 2016-01-01 -o sample_objects.json
```

For more information about the data format of objects and locations, see Data Formats section below.

### List mode

The list mode lists Az and El (also RA and Dec) of objects in a terminal.
To list Az and El of solar objects seen from Mitaka in real time, for example:

```bash
$ ./pazel -m list -l mitaka tokyo
```

As a result:

```
------------------------------------------------------------------------------
|                             Pazel's list mode                              |
------------------------------------------------------------------------------
|                |    name        Tokyo                                      |
|    Observer    |    address     Tokyo, Japan                               |
|    Location    |    lat, lng    35.6894875, 139.6917064                    |
|                |    timezone    Japan Standard Time (UTC+9.0)              |
------------------------------------------------------------------------------
|                |                   |                   |                   |
|      Time      |    LT 00:05:03    |   UTC 15:05:03    |   LST 16:58:02    |
|                |                   |                   |                   |
------------------------------------------------------------------------------
|     Object     |            RA           Dec            Az            El   |
------------------------------------------------------------------------------
|      Sun       |    4:31:49.96    21:53:12.7     7:10:47.4   -32:05:24.4   |
|    Mercury     |    3:01:35.28    13:13:08.8    34:47:01.6   -33:52:25.8   |
|     Venus      |    4:23:38.73    21:17:28.3     9:30:06.1   -32:26:23.6   |
|      Mars      |   15:45:24.99   -21:27:17.3   199:37:57.8    30:21:03.5   |
|    Jupiter     |   11:02:44.51     7:31:05.1   275:26:27.0     5:28:32.9   |
|     Saturn     |   16:48:53.69   -20:36:33.3   182:34:18.9    33:41:00.1   |
|     Uranus     |    1:26:26.48     8:25:39.8    59:22:15.1   -23:31:24.1   |
|    Neptune     |   22:54:49.32    -7:50:03.4    96:50:26.1    -2:27:38.1   |
------------------------------------------------------------------------------
```

To get a notification when El of the objects whose names begin with `!` are below 30 deg:

```bash
$ ./pazel -m list -l mitaka tokyo --lim 30: --bell
```

## JSON Data format

Pazel loads and stores the information about objects and locations in JSON files.
Currently two types of data formats are used: objects and locations.

### Objects

The information about objects must be stored in a JSON file.
Objects to be plotted are described as `key: parameters` like the following JSON format:

```json
{
    "Sun": "Sun",
    "Moon": "Moon",
    "!Ori-KL": {"ra": "05:35:14.50", "dec": "-05:22:30.4", "epoch": "J2000"},
    "#M17-SW": {"ra": "18:20:19.68", "dec": "-16:13:31.5", "epoch": "J2000"}
}
```

#### key

Key is a name of the object and used as a label in a plot.
The following two special characters which with key begins control objects in a plot:

+ `#name`: ignore the object to be plotted
+ `!name`: activate the notification (in list mode only)

#### parameters

If you want to create your own location, the following parameters are required:

+ for solar objects
    + `object name`: such as Sun, Jupiter, Moon, etc
+ for user defined objects
    + `ra` (required): right ascension (HH:MM:SS.SS)
    + `dec` (required): declination (DD:MM:SS.S)
    + `epoch` (optional): J2000 (default), B1950, or B1900

### Locations

The information about locations is stored in `known_locations.json`.
It is automatically created for the first time you execute Pazel.
Locations are described as `key: parameters` like the following JSON format:

```json
{
    "mitaka+tokyo": {
        "name": "Mitaka",
        "address": "Mitaka, Tokyo, Japan",
        "place_id": "ChIJgwNoZdDvGGARvkvyBJgEeCw",
        "lat": 35.683513,
        "lng": 139.5595842,
        "tz_name": "Japan Standard Time",
        "tz_value": 9.0
    }
}
```

#### key

If you spacify `-l mitaka tokyo` as arguments, for example, key will be `mitaka+tokyo`.
You can modify it like `preset1` and then spacify `-l preset1` for the next time you execute Pazel.

#### parameters

If you want to create your own location, the following parameters are required or optional:

+ `name` (required): location's short name
+ `address` (optional): location's full address
+ `place_id` (optional): Google Maps place ID
+ `lat` (required): latitude in degree
+ `lng` (required): longitude in degree
+ `tz_name` (required): timezone's name
+ `tz_value` (required): timezone's value in hour
