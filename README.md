# Skyfield almanac for WeeWX

Calculate solar, lunar, planetary and optional satellite/star almanac values
for WeeWX templates using Skyfield and downloaded astronomical data.

This repository mirrors [the upstream source](https://github.com/roe-dl/weewx-skyfield-almanac). The maintained
catalog archive is **0.7.post1**, carrying the complete GPL license text. Install that fixed archive below;
the mirrored default branch can differ from its repaired payload.

The catalog archive targets WeeWX `>=5.5.2,<6` and Python `>=3.10,<3.15`.


## Contents

* [Why should I use this extension?](#why-should-i-use-this-extension)
* [Prerequisites](#prerequisites)
* [Installation instructions](#installation-instructions)
* [Configuration instructions](#configuration-instructions)
* [Usage](#usage)
* [Help! Report creation gets very slow](#help-report-creation-gets-very-slow)
* [Customization of WeeWX using this extension](#customization-of-weewx-using-this-extension)
* [Maps, Star charts and other images](#maps)
* [PyEphem and Skyfield](#pyephem-and-skyfield)
* [Live data](#live-data)
* [For developers](#for-developers)
* [FAQ](#faq)
* [Credits](#credits)
* [Links](#links)

## Why should I use this extension?

Skyfield supplies an alternative almanac provider for existing WeeWX skins.
The extension uses JPL ephemeris files and adds attributes described in the
[customization guide](Customization.md). Existing `$almanac` tags use this
provider once initialization finishes; PyEphem remains a fallback when enabled.

## Prerequisites

The source requires WeeWX 5.3 or later. Use the catalog compatibility range
above for the maintained archive. Install Skyfield, NumPy and Requests in the
Python environment running WeeWX. Pandas is optional for loading the Hipparcos
star catalog, and is needed when you want those stars on a sky map.

Activate the virtual environment that runs WeeWX before using `python` or
`weectl`, then install the runtime packages:

```sh
python -m pip install --only-binary=:all: 'skyfield>=1.49' 'numpy' 'requests'
```

For Hipparcos stars, also install the optional package:

```sh
python -m pip install --only-binary=:all: 'pandas'
```

These commands use wheels and require no native build tools. They require
wheels for your Python version and platform. For a system installation, use
equivalent distribution packages in WeeWX's Python environment where available.
Skyfield 1.49 or later is needed for the non-TLE satellite support. The extension
installer does not install these Python packages.

The service needs Internet access for its configured JPL kernels and optional
IERS, Hipparcos and satellite data, plus a writable `skyfield` directory below
the configured SQLite data root. Large ephemeris files can delay initialization.

## Installation instructions

Replace `/path/to/weewx.conf` with your existing station configuration. Use
`weectl` from the Python environment running WeeWX to install the archive:

```sh
weectl extension install https://github.com/weewx-packages/catalog/releases/download/skyfield-almanac/v0.7.post1/skyfield-almanac-0.7.post1.tar.gz --config /path/to/weewx.conf
```

You can also download that URL and pass the local archive to the same command.
Use elevated permissions only when your station files require them.

The installer registers `user.skyfieldalmanac.SkyfieldService` as a prep service
and `user.skyfieldalmanac.LiveService` as a data service. It also adds missing
localization keys to existing skins' language files; see [language files](lang/README.md).
To skip that step, append `--skip-localization` to the installation command.
To preload the configured ephemeris and timescale files, append
`--download-ephemeris`. These are custom arguments handled by this installer.

Restart WeeWX yourself. Check its log for ephemeris initialization and let a
report cycle finish. A small explicit configuration is:

```ini
[Almanac]
    [[Skyfield]]
        enable = true
        ephemeris = de440s.bsp
        use_builtin_timescale = true
```

The installer sets `de440.bsp`; this smaller example selects `de440s.bsp`.
Other configured planetary constants and optional catalogs can still require
downloads. For an offline station, populate the required files before removing
network access. `--download-ephemeris` does not preload every optional catalog.

In a Cheetah template, existing tags such as `$almanac.sun.rise` use Skyfield
after initialization. The examples and reference below cover additional tags.

## Configuration instructions

There is no need to configure anything, but there are some tuning options
available if you have special requirements. See the customization guide,
section
[configuration](./Customization.md#configuration) for details.

## Usage

There is no other requirement than installing this extension to replace
PyEphem calculated values by Skyfield calculated values in existing skins.

Once the weewx-skyfield-almanac extension is installed and initialized after
startup, `$almanac.hasExtras` becomes `True` and extended almanac
information is available. Initialization can take several archive
intervals to be completed at first run after installation, depending on 
configuration.

Look at 
[weewx-skymap examples](https://github.com/weewx-packages/weewx-skymap-almanac/tree/master/examples/Seasons)
for a replacement almanac page for the
WeeWX built-in Seasons skin to show additional values provided by this
extension.

## Help! Report creation gets very slow

If you encounter speed issues, here are some hints:

* To check if this extension is part of the problem you can set 
  `enable=false` in section `[Almanac]`, sub-section `[[Skyfield]]`, 
  in `weewx.conf`. That switches it off and switches the WeeWX built-in
  PyEphem based almanac on. Make sure, you restarted WeeWX after editing
  `weewx.conf`.
* In case you use the 
  [weewx-GTS extension](https://github.com/roe-dl/weewx-GTS)
  you need at least version 1.2 of it together with this extension.
* Calculating periods of visibility can slow down report creation.
  See section [`genVisibleTimespans()`](Customization.md#genvisibletimespans)
  for details and solutions.
* For some attributes `almanac_time` can take a list of timestamps.
  Then, a list of respective results is returned. Using this can speed
  up things.

## Customization of WeeWX using this extension

If you want to extend your skin by additional almanac data, see the
[customization guide](./Customization.md) of this extension as well
as the WeeWX customization guide, section "The Cheetah generator",
sub-section
"[Almanac](http://weewx.com/docs/latest/custom/cheetah-generator/#almanac)",
for a detailed description how to use the almanac in WeeWX. 

## Maps

If you want to draw sky maps you may want to install the
[WeeWX sky map extension](https://github.com/roe-dl/weewx-skymap-almanac).

## PyEphem and Skyfield

If you install both PyEphem and Skyfield, Skyfield is preferred. If the
given heavenly body is available with Skyfield, the attribute is calculated
using Skyfield. Otherwise PyEphem is tried. If neither Skyfield nor 
PyEphem know about the body, an exception is raised.

## Live data

Some skins do not only update the web pages once every archive interval
but present live data out of LOOP packets. This extension can add fast 
changing almanac values to the LOOP packet for live updates. To use them
include them in MQTT output. To activate live data, set the
`enable_live_data` configuration option to `true`, which is the default.
For details see [LOOP live data](LOOP-live-data.md) guide.

## For developers

Not only the WeeWX almanac itself is extensible. If you are a developer
you can also write a WeeWX extension, that adds new attributes to heavenly
bodies covered by this extension. To do so write an almanac extension
and append a reference to it to `user.skyfieldalmanac.subalmanacs`.

Within your extension you need to define a class based on
`weewx.almanac.AlmanacType` with a method 
`get_almanac_data(self, almanac_obj, attr)` in it.
`attr` passes the attribute out of the template. For all attributes
you want to support, do the calculation and return a `ValueHelper`.
For attributes you do not support, raise `weewx.UnknownType(attr)`.
`almanac_obj` provides the following information:
* `almanac_obj.heavenly_body`: the name of the heavenly body. 
* `almanac_obj.almanac.time_ts`: the timestamp `$almanac` is bound to.
  It may be a list, in which case you have to do your calculation
  for all the timestamps provided.
* `almanac_obj.almanac.lat`: latitude
* `almanac_obj.almanac.lon`: longitude
* `almanac_obj.almanac.texts`: the `[Almanac]` section out of the language
  file

## FAQ

Q: I installed the Skyfield module, but no extended almanac values
   are displayed.

A: Depending on the ephemeris file you chose initialization takes more or
   less time. While it is not finished, the skin displays some core
   values only. Wait a moment, and the extended values will show up.
   If the ephemeris file is huge and it is the first run after
   installation, it can take several archive intervals to be completed.


Q: Which ephemeris file should I use?

A: The ephemeris files differ in size and coverage of heavenly bodies and
   time. The runtime fallback is `de440s.bsp`, while the installer sets `de440.bsp`. It
   covers Sun, Earth, Moon, and some data of the long known planets. If
   you need more, try `de440.bsp` or even `de441.bsp`.
   From time to time the NASA releases new sets of ephemeris files. Then you
   can try the new edition.
   See [Planets and their moons: JPL ephemeris files](https://rhodesmill.org/skyfield/planets.html)
   for more details.


Q: Are there disadvantages of Skyfield?

A: Yes. Always are. Skyfield depends on NumPy, while PyEphem does not.


Q: How do I have to adapt my skin to use this extension?

A: There is nothing to do. Installing this extension is enough. But you
   could add the additional attributes supported by this extension
   to your skin to display them there.


Q: I installed a skin and it does not show the localized names of planets
   and constellations.

A: You will have to re-install this extension after you installed the
   skin. 


Q: I get the log message "ERROR user.skyfieldalmanac: live almanac error:
   'mars'" What can I do?

A: Try "mars_barycenter" instead of "mars".


Q: What can I do if I don't want the skins' language files augmented with
   astronomic terms from this extension?

A: Append `--skip-localization` to the fixed-archive installation command.
   The installer recognizes this custom argument and skips language-file changes.

## Credits

* [Tom Keffer et al. (WeeWX)](https://github.com/weewx/weewx)
* [Brandon Rhodes (Skyfield)](https://github.com/skyfielders/python-skyfield)

## Links

* [WeeWX](https://weewx.com)
* [Skyfield](https://rhodesmill.org/skyfield/)
* [WeeWX sky map extension](https://github.com/roe-dl/weewx-skymap-almanac)
* [International Earth Rotation and Reference Service IERS](https://iers.org)
  (provides the timescale file `finals2000A.all`)
* [Jet Propulsion Laboratory JPL](https://www.jpl.nasa.gov)
  (provides the ephemeris files)
* [JPL: SPK files](https://naif.jpl.nasa.gov/pub/naif/generic_kernels/spk/)
* [JPL: Get small body SPK files](https://ssd.jpl.nasa.gov/horizons/app.html#/)
* [Issue #981: PyEphem is deprecated](https://github.com/weewx/weewx/issues/981)
* [International Terrestrial Reference System ITRS](https://en.wikipedia.org/wiki/International_Terrestrial_Reference_System_and_Frame)
* [ESA's Hipparcos mission and catalog](https://www.cosmos.esa.int/web/hipparcos/home)
* [The brightest stars in the Hipparcos catalogue](https://www.cosmos.esa.int/web/hipparcos/brightest)
* [CelesTrak](https://celestrak.org)
* [Andrea K. Myers-Beaghton et al.: The moon tilt illusion](https://www.seas.upenn.edu/~amyers/MoonPaperOnline.pdf)
* [Karlheinz Schott (&dagger;): "Falsche" Mondneigung - Wohin zeigt die Mondsichel?](https://falsche-mondneigung.jimdofree.com/b-geometrische-darstellung-und-berechnung/) (german)
* [Lunar standstill (Wikipedia)](https://en.wikipedia.org/wiki/Lunar_standstill)

## License and origin

Copyright 2025-2026 Johanna Karen Roedenbeck and credited contributors.
[GNU GPLv3 or later](LICENSE). The mirrored upstream source and its references
remain available; the catalog `.post1` archive includes the complete license.
Astronomical data coverage depends on the kernels and catalogs you configure.
