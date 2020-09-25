tess-tiler
==========

**Turns big FFI data into small FFI data.**

*tess-tiler* is a fast and memory-efficient package designed to convert collections of TESS Full Frame Images (FFI) into sub-sections which can fit into memory. This package uses a pre-defined tiling scheme which allows users to refer to a sub-sections of FFIs in a standardized way.  It is inspired by the `map tiling scheme <https://www.maptiler.com/google-maps-coordinates-tile-bounds-projection/>`_ in use by Google Maps.

The *tess-tiler* tiling scheme
------------------------------

Each tile is uniquely identified by six parameters:


.. code-block:: python

    >>> import tess_tiler as tt
    >>> tile = tt.TessTile(sector, camera, ccd, zoom, x, y, margin=0)

These parameters are defined as follows:

* ``sector``, ``camera``, and ``ccd`` are the standard TESS full-frame image identifiers.
* The ``zoom`` level is defined such that an FFI fits entirely into a single tile at zoom level 0. All subsequent zoom levels divide the image into a grid of 2^zoom x 2^zoom tiles.
* The (x, y) parameters identify the position of the tile along the column and row direction of the FFI. Their values range between 0 and 2^zoom - 1.  Tile (x, y) = (0, 0) is the bottom left corner of the FFI.
* An optional ``margin`` parameter can be specified to allow tiles to overlap by a specified number of pixels.


Examples
~~~~~~~~

==========================  =================
tile definition             description
==========================  =================
TessTile(1, 2, 3, 0, 0, 0)  Full FFI for Sector #1, Camera #2, CCD #3.
TessTile(1, 2, 3, 1, 0, 0)  Bottom-left quadrant (1/4th) of the same FFI.
TessTile(1, 2, 3, 1, 1, 1)  Top-right quadrant.
TessTile(1, 2, 3, 2, 2, 0)  Bottom-right octant (1/8th).
TessTile(1, 2, 3, 2, 0, 2)  Top-left octant.
==========================  =================

Memory requirements
~~~~~~~~~~~~~~~~~~~

The memory required to store a single tile as a function of the zoom level is as follows: 

========== ========= ============== ========== ============
zoom level tiles/ccd tile_size (px) sky area   bytes/sector
========== ========= ============== ========== ============
0            1       2048 x 2048    143 deg²   64 GB
1            4       1024 x 1024    36 deg²    16 GB
2            16      512 x 512      9 deg²     4 GB
3            32      256 x 256      2 deg²     1 GB
4            64      128 x 128      0.6 deg²   256 MB
5            128     64 x 64        0.1 deg²   64 MB
6            256     32 x 32        0.04 deg²  16 MB
========== ========= ============== ========== ============

In this table, bits/sector denotes the amount of memory required to cut out 27 days worth of 10-minute FFI data at 32 bits per pixel.


Example use
-----------

*tess-tiler* provides a fast and memory-efficient way to extract the image data for an entire sector of FFIs, without utilizing more than the memory required by the specific tile. 

.. code-block:: python

    >>> import tess_tiler as tt
    >>> tile = tt.TessTile(sector=1, camera=2, ccd=3, zoom=4, x=2, y=3)
    >>> loader = tt.TileLoader(ffi_filenames)
    >>> tpf = loader.create_tpf(tile)

To create a list of ``TessTile`` objects that cover a specific coordinate in the sky, you can use the sister *tess-fov* package:

.. code-block:: python

    >>> import tess_fov as tf
    >>> tiles = tf.radec_to_tiles(ra, dec, zoom=3)  # returns a list of TessTile objects

To obtain the parameters of all the tiles that observed a known Solar System object, you can use the sister *tess-ephem* package:

.. code-block:: python

    >>> import tess_ephem as te
    >>> tiles = te.asteroid_to_tiles("Vesta", zoom=2)  # returns a list of TessTile objects


How does *tess-tiler* compare to other tools?
---------------------------------------------
Several excellent tools already exist which focus on enabling users to extract or download sub-regions from FFI images via the web, for example:

* ``TessCut`` enables users to download regions up to 100x100 pixels in size centered on an arbitrary coordinate, utilizing pre-computed data cubes stored in the cloud. In contrast, *tess-tiler* focuses on efficiently extracting tiles from a locally-stored collection of FFI files, according to a pre-defined tiling scheme.

* ``eleanor`` enables users to create or download 104-by-148 pixel cutout regions called postcards. *tess-tiler* expands this concept by providing a generic tool to create such postcards at different zoom levels from a local collection of FFI files.
