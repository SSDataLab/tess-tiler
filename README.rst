tess-tiler
==========

**Turns Big FFI Data into Small FFI Data.**

*tess-tiler* is a fast and memory-efficient package intended to turn collections of TESS Full Frame Images (FFI) into smaller tiles which can fit more easily into memory. This package uses a pre-defined tiling scheme which allows users to refer to a sub-set of FFI data in a standardized way, inspired by the `tiling scheme <https://www.maptiler.com/google-maps-coordinates-tile-bounds-projection/>`_ in use by Google Maps.

The *tess-tiler* tiling scheme
------------------------------

Each tile is uniquely identified by a tuple of six numbers ``(sector, camera, ccd, zoom, x, y)`` defined as follows:

* ``sector``, ``camera``, and ``ccd`` are the standard TESS full-frame image identifiers.
* The ``zoom`` level is defined such that an FFI fits entirely into a single tile at ``zoom=0``. All subsequent zoom levels divide the image into a grid of 2^zoom x 2^zoom tiles.
* The (x, y) coordinates identify the position of the tile along the column and row direction of the FFI. The x and y values range between 0 and 2^zoom - 1.  Tile (x, y) = (0, 0) is located in the bottom left of the FFI.
* An optional `margin` parameter can be specified to allow tiles to overlap by a specified number of pixels.


For example:

================== =================
tile_identifier    description
================== =================
(1, 2, 3, 0, 0, 0)   Full FFI for Sector #1, Camera #2, CCD #3.
(1, 2, 3, 1, 0, 0)   Bottom-left quadrant (1/4th) of the same FFI.
(1, 2, 3, 1, 1, 1)   Top-right quadrant.
(1, 2, 3, 2, 2, 0)   Bottom-right octant (1/8th).
(1, 2, 3, 2, 0, 2)   Top-left octant.
================== =================

This tiling scheme provides a convenient way to work with sub-sections of FFI images. The number of tiles and memory requirements as a function of the zoom level are as follows: 

========== ========= ============== ========== ===========
zoom level tiles/ccd tile_size (px) sky area   bits/sector
========== ========= ============== ========== ===========
0            1       2048 x 2048    143 deg²   64 GB
1            4       1024 x 1024    36 deg²    16 GB
2            16      512 x 512      9 deg²     4 GB
3            32      256 x 256      2 deg²     1 GB
4            64      128 x 128      0.6 deg²   256 MB
5            128     64 x 64        0.1 deg²   64 MB
6            256     32 x 32        0.04 deg²  16 MB
========== ========= ============== ========== ===========

In this table, bits/sector denotes the amount of memory required to cut out 27 days worth of 10-minute FFI data at 32 bits per pixel.


Example use
-----------

The `tess-tiler` package provides a fast and memory-efficient way to extract the image data for an entire sector of FFIs, without utilizing more than the memory required by the specific tile. 

.. code-block:: python

    >>> import tess_tiler as tt
    >>> loader = tt.TileLoader(ffi_filenames)
    >>> tpf = loader.get_tpf(zoom=4, x=2, y=3)

To obtain the tile that covers a specific coordinate in the sky, you can use the sister *tess-fov* package:

.. code-block:: python

    tess_fov as tf
    fov = tf.TessFov()
    tiles = fov.radec_to_tiles(ra, dec)


How does `tess-tiler` compare to other tools?
---------------------------------------------
Several excellent tools already exist which focus on enabling users to extract or download sub-regions from FFI images via the web, for example:

* ``TessCut`` enables users to download regions up to 100x100 pixels in size centered on an arbitrary coordinate, utilizing pre-computed data cubes stored in the cloud. In contrast, *tess-tiler* focuses on efficiently extracting tiles from a locally-stored collection of FFI files, according to a pre-defined tiling scheme.

* ``eleanor`` enables users to create or download 104-by-148 pixel cutout regions called postcards. `tess-tiler` expands this concept by providing a generic tool to create such postcards at different zoom levels from a local collection of FFI files.
