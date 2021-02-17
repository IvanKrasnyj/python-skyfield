
====================================
 Planets, and Choosing an Ephemeris
====================================

If you are interested in observing the planets,
the Jet Propulsion Laboratory (JPL)
offers high accuracy tables of planet positions
over time spans ranging from decades to centuries.
You can use ``load()`` to download a JPL ephemeris:

.. testsetup::

   __import__('skyfield.tests.fixes').tests.fixes.setup()

.. testcode::

    from skyfield.api import load
    planets = load('de421.bsp')

See :doc:`files` to learn more about downloading files.

Here are several popular ephemerides that you can use with Skyfield.
The filenames matching ``de*``
predict the positions of many or all of the major planets,
while ``jup310.bsp`` focuses on Jupiter and its major moons:

==========  ====== =============== ====== ============ ========
File         Size        Years     Issued Moon         Inner
==========  ====== =============== ====== ============ ========
de405.bsp    63 MB   1600 to 2200  1997
de406.bsp   287 MB  −3000 to 3000  1997
de421.bsp    17 MB   1900 to 2050  2008
de422.bsp   623 MB  −3000 to 3000  2009
de430t.bsp  128 MB   1550 to 2650  2013   0″.0005 – 1″ 0″.0002
de431t.bsp  3.5 GB –13200 to 17191 2013
jup310.bsp  932 MB   1900 to 2100  2013
==========  ====== =============== ====== ============ ========

You can think of negative years, as cited in the above table,
as being almost like years BC except that they are off by one.
Historians invented our calendar back before zero was a counting number,
so AD 1 was immediately preceded by 1 BC without a year in between.
But astronomers count backwards AD 2, AD 1, 0, −1, −2, and so forth.

So if you are curious about the positions of the planets back in 44 BC,
when Julius Caesar was assassinated,
be careful to ask an astronomer about the year −43 instead.

A popular choice of ephemeris is DE421.
It is recent, has good precision,
was designed for general-purpose use,
and is only 17 MB in size:

Once an ephemeris file has been downloaded to your current directory,
re-running your program will simply reuse the copy on disk
instead of downloading it all over again.

After you have loaded an ephemeris and have used a statement like:

.. testcode::

    mars = planets['Mars']

— to retrieve a planet, consult the chapter :doc:`positions`
to learn about all the positions that you can use it to generate.

If you want to examine the segments that make up the ephemeris,
you can loop over its ``segments`` list.
You can ``print()`` a segment to see a textual description,
or access segment attributes that give its center, target,
and the dates over which it provides valid positions:

.. testcode::

    ts = load.timescale()

    segment = planets.segments[0]
    start, end = segment.time_range(ts)

    print('Center:', segment.center_name)
    print('Target:', segment.target_name)
    print('Date range:', start.tdb_strftime(), '-', end.tdb_strftime())

.. testoutput::

    Center: 0 SOLAR SYSTEM BARYCENTER
    Target: 1 MERCURY BARYCENTER
    Date range: 1899-07-29 00:00:00 TDB - 2053-10-09 00:00:00 TDB

For example, you can see above
that the first segment of the ephemeris DE421
provides the position of Mercury relative to the center of the Solar System
over the entire twentieth century and half of the twenty-first.

Making an excerpt of an ephemeris
=================================

Several of the ephemeris files listed below are very large.
While most programmers will follow the example above and use DE421,
if you wish to go beyond its 150-year period
you will need a larger ephemeris.
And programmers interested in the moons of Jupiter
will need JUP310, which weighs in at nearly a gigabyte.

What if you need data from a very large ephemeris,
but don’t require its entire time span?

When you installed Skyfield another library named ``jplephem``
will have been installed.
When invoked from the command line,
it can build an excerpt of a larger ephemeris
without needing to download the entire file,
thanks to the fact that HTTP supports a ``Range:`` header
that asks for only specific bytes of a file.
For example,
let’s pull two weeks of data for Jupiter’s moons
(using a shell variable ``$u`` for the URL
only to make the command less wide here on the screen
and easier to read)::

$ u=https://naif.jpl.nasa.gov/pub/naif/generic_kernels/spk/satellites/jup310.bsp
$ python -m jplephem excerpt 2018/1/1 2018/1/15 $u jup_excerpt.bsp

The resulting file ``jup_excerpt.bsp`` weighs in
at only 0.2 MB instead of 932 MB
but supports all of the same objects as the original JUP310
over the given two-week period::

  $ python -m jplephem spk jup_excerpt.bsp
  File type DAF/SPK and format LTL-IEEE with 13 segments:
  2458119.75..2458210.50  Jupiter Barycenter (5) -> Io (501)
  2458119.50..2458210.50  Jupiter Barycenter (5) -> Europa (502)
  2458119.00..2458210.50  Jupiter Barycenter (5) -> Ganymede (503)
  2458119.00..2458210.50  Jupiter Barycenter (5) -> Callisto (504)
  ...

You can load and use it directly off of disk
with :func:`~skyfield.iokit.load_file()`.

How segments are linked to predict positions
============================================

You can ``print()`` an ephemeris to learn which objects it supports.

.. testcode::

    print(planets)

.. testoutput::

    SPICE kernel file 'de421.bsp' has 15 segments
      JD 2414864.50 - JD 2471184.50  (1899-07-28 through 2053-10-08)
          0 -> 1    SOLAR SYSTEM BARYCENTER -> MERCURY BARYCENTER
          0 -> 2    SOLAR SYSTEM BARYCENTER -> VENUS BARYCENTER
          0 -> 3    SOLAR SYSTEM BARYCENTER -> EARTH BARYCENTER
          0 -> 4    SOLAR SYSTEM BARYCENTER -> MARS BARYCENTER
          0 -> 5    SOLAR SYSTEM BARYCENTER -> JUPITER BARYCENTER
          0 -> 6    SOLAR SYSTEM BARYCENTER -> SATURN BARYCENTER
          0 -> 7    SOLAR SYSTEM BARYCENTER -> URANUS BARYCENTER
          0 -> 8    SOLAR SYSTEM BARYCENTER -> NEPTUNE BARYCENTER
          0 -> 9    SOLAR SYSTEM BARYCENTER -> PLUTO BARYCENTER
          0 -> 10   SOLAR SYSTEM BARYCENTER -> SUN
          3 -> 301  EARTH BARYCENTER -> MOON
          3 -> 399  EARTH BARYCENTER -> EARTH
          1 -> 199  MERCURY BARYCENTER -> MERCURY
          2 -> 299  VENUS BARYCENTER -> VENUS
          4 -> 499  MARS BARYCENTER -> MARS

Bodies in JPL ephemeris files are each identified by an integer,
but Skyfield translates them so that you do not have to remember
that a code like 399 stands for the Earth and 499 for Mars.

Each ephemeris segment predicts the position of one body
with respect to another.
Sometimes several segments sometimes have to be combined
to generate a complete position.
The DE421 ephemeris shown above, for example,
can produce the position of the Sun directly.
But if you ask it for the position of Earth
then it will have to add together two distances:

* From the Solar System’s center (0) to the Earth-Moon barycenter (3)
* From the Earth-Moon barycenter (3) to the Earth itself (399)

This happens automatically behind the scenes.
All you have to say is ``planets[399]`` or ``planets['Earth']``
and Skyfield will put together a solution using the segments provided.

.. testcode::

    earth = planets['earth']
    print(earth)

.. testoutput::

    Sum of 2 vectors:
     'de421.bsp' segment 0 SOLAR SYSTEM BARYCENTER -> 3 EARTH BARYCENTER
     'de421.bsp' segment 3 EARTH BARYCENTER -> 399 EARTH

Each time you ask this ``earth`` object for its position at a given time,
Skyfield will compute both of these underlying vectors
and add them together to generate the position.

Closing the file automatically
==============================

If you need to close files as you finish using them
instead of waiting until the application exits,
each Skyfield ephemeris offers a
:meth:`~skyfield.jpllib.SpiceKernel.close()` method.
It can either be called manually when you are done with an ephemeris,
or you can use Python’s |closing|_ context manager
to call the method automatically
at the completion of a ``with`` statement:

.. |closing| replace:: ``closing()``
.. _closing: https://docs.python.org/3/library/contextlib.html#contextlib.closing

.. testcode::

    from contextlib import closing

    ts = load.timescale()
    t = ts.J2000

    with closing(planets):
        planets['venus'].at(t)  # Ephemeris can be used here

    planets['venus'].at(t)  # But it’s closed outside the “with”

.. testoutput::

    Traceback (most recent call last):
      ...
    ValueError: seek of closed file

.. testcleanup::

   __import__('skyfield.tests.fixes').tests.fixes.teardown()

.. _third-party-ephemerides:

Type 1 and Type 21 ephemeris formats
====================================

If you generate an ephemeris with a tool like NASA’s
`HORIZONS <https://ssd.jpl.nasa.gov/horizons.cgi>`_ system,
it might be in a format not yet natively supported by Skyfield.
The first obstacle to opening the ephemeris
might be its lack of a recognized suffix:

.. testcode::

    load('wld23593.15')

.. testoutput::

    Traceback (most recent call last):
      ...
    ValueError: Skyfield does not know how to open a file named 'wld23593.15'

A workaround for the unusual filename extension
is to open the file manually using Skyfield’s JPL ephemeris support.
The next obstacle, however, will be a lack of support
for Type 21 ephemerides in Skyfield:

.. testcode::

    from skyfield.jpllib import SpiceKernel
    kernel = SpiceKernel('wld23593.15')

.. testoutput::

    Traceback (most recent call last):
      ...
    ValueError: SPK data type 21 not yet supported

Older files with a similar format
might instead generate the complaint
“SPK data type 1 not yet supported.”

Happily, thanks to Shushi Uetsuki,
a pair of third-party libraries exist
that offer preliminary support for Type 1 and Type 21 ephemerides!

* https://pypi.org/project/spktype01/
* https://pypi.org/project/spktype21/

Their documentation already includes examples of generating raw coordinates,
but many Skyfield users will want to use them
in conjunction with standard Skyfield methods like ``observe()``.
To integrate them with the rest of Skyfield,
you will want to define a new vector function class
that calls the third-party module to generate coordinates:

.. testcode::

    from skyfield.constants import AU_KM
    from skyfield.vectorlib import VectorFunction
    from spktype21 import SPKType21

    t = ts.utc(2020, 6, 9)

    eph = load('de421.bsp')
    earth = eph['earth']

    class Type21Object(VectorFunction):
        def __init__(self, kernel, target):
            self.kernel = kernel
            self.center = 0
            self.target = target

        def _at(self, t):
            k = self.kernel
            r, v = k.compute_type21(0, self.target, t.whole, t.tdb_fraction)
            return r / AU_KM, v / AU_KM, None, None

    kernel = SPKType21.open('wld23593.15')
    chiron = Type21Object(kernel, 2002060)

    ra, dec, distance = earth.at(t).observe(chiron).radec()
    print(ra)
    print(dec)

.. testoutput::

    00h 27m 38.99s
    +05deg 57' 08.9"

Hopefully this third-party support
for Type 1 and Type 23 SPK ephemeris segments
will be sufficient for projects that need them,
until there is time for a Skyfield contributor
to integrate such support into Skyfield itself.

Ephemeris bibliography
======================

DE405 / DE406

* `JPL Planetary and Lunar Ephemerides, DE405/LE405
  <ftp://ssd.jpl.nasa.gov/pub/eph/planets/ioms/de405.iom.pdf>`_
  (Standish 1998)

* `Check on JPL DE405 using modern optical observations
  <https://aas.aanda.org/articles/aas/pdf/1998/18/ds1546.pdf>`_
  (Morrison and Evans 1998)

* `CCD Positions for the Outer Planets in 1996–1997
  Determined in the Extragalactic Reference Frame
  <https://iopscience.iop.org/article/10.1086/300507/fulltext/>`_
  (Stone 1998)

* `Astrometry of Pluto and Saturn
  with the CCD meridian instruments of Bordeaux and Valinhos
  <https://www.aanda.org/articles/aa/full/2002/09/aa1965/aa1965.html>`_
  (Rapaport, Teixeira, Le Campion, Ducourant1, Camargo,
  Benevides-Soares 2002)

DE421

* `The Planetary and Lunar Ephemeris DE421
  <https://ipnpr.jpl.nasa.gov/progress_report/42-178/178C.pdf>`_
  (Folkner, Williams, Boggs 2009)

DE430 / DE431

* `The Planetary and Lunar Ephemerides DE430 and DE431
  <https://ipnpr.jpl.nasa.gov/progress_report/42-196/196C.pdf>`_
  (Folkner, Williams, Boggs, Park, Kuchynka 2014)

  » *Very long baseline interferometry measurements of spacecraft at
  Mars allow the orientation of the ephemeris to be tied to the
  International Celestial Reference Frame with an accuracy of
  0″.0002. This orientation is the limiting error source for the orbits
  of the terrestrial planets, and corresponds to orbit uncertainties of
  a few hundred meters.*

  » *The DE431 time span from the year –13,200 to the year 17,191
  extends far beyond historical times and caveats are offered.  For the
  planets, uncertainties in the initial conditions of the orbits will
  cause errors in the along-track directions that increase at least
  linearly with time away from the present. … For the Moon, the
  uncertainty given for the tidal acceleration causes a 28 m/century²
  along-track uncertainty*

* `DE430 Lunar Orbit, Physical Librations and Surface Coordinates
  <https://naif.jpl.nasa.gov/pub/naif/generic_kernels/spk/planets/de430_moon_coord.pdf>`_
  (Williams, Boggs, Folkner 2013)

  » *The right ascension and declination differences reach up to ~1 m
  perpendicular to the radius, or up to ~1/2 milliarcsecond (mas) in
  angle … over the 1970-2012 span of lunar laser ranging data. DE430
  agrees well with DE421.*

  » *From a comparison of different integrations, it appears that the
  error in the DE430 orbit at 1800 is ~0.1″ and the error at 1600 is
  ~1″*

.. Also ftp://ssd.jpl.nasa.gov/pub/eph/planets/ioms/de434.iom.pdf

.. Also ftp://ssd.jpl.nasa.gov/pub/eph/planets/ioms/de435.iom.pdf

.. Also https://naif.jpl.nasa.gov/pub/naif/JUNO/kernels/spk/de438s.bsp.lbl

Analysis mentioning several ephemerides

* `Modeling the Uncertainties of Solar-System Ephemerides
  for Robust Gravitational-Wave Searches with Pulsar Timing Arrays
  <https://arxiv.org/pdf/2001.00595.pdf>`_
  (The NANOGrav Collaboration 2020)

  » *Crucially, the ephemerides do not generally provide usable error
  representations. … Our model complements published SSEs [Solar System
  Ephemerides], which do not generally include usable time-domain
  representations of orbit uncertainties.*
