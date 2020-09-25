tess-tiler
==========

**Turns Big FFI Data into Small FFI Data.**

*tess-tiler* is a user-friendly package intended to turn big sets of TESS Full Frame Images (FFI) into smaller tiles which can fit more easily into memory. The package uses a pre-defined tiling scheme which allows users to refer to a sub-set of FFI data in a convenient and well-defined way, inspired by the tiling scheme adopted by Google Maps.

The *tess-tiler* tiling scheme
------------------------------

Each tile is uniquely identified by a tuple of six numbers, ``(sector, camera, ccd, zoom, x, y)``, which are defined as follows:

* ``sector``, ``camera``, and ``ccd`` are the standard TESS full-frame image identifiers.
* The ``zoom`` level is defined such that an FFI fits entirely into a single tile at ``zoom=0``. All subsequent zoom levels divide the image into a grid of 2^zoom x 2^zoom tiles.
* The (x, y) coordinates identify the position of the tile along the column and row direction of the FFI, ranging between 0 and 2^zoom - 1.  Tile (x, y) = (0, 0) is located in the bottom left of the FFI.


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

This tiling scheme provides a convenient way to work with small or large sub-sections of FFI images. The number of tiles and memory requirements as a function of the zoom level are as follows: 

========== ========= ============== ===========
zoom level tiles/ccd tile_size (px) bits/sector
========== ========= ============== ===========
0            1       2048 x 2048     64 GB
1            4       1024 x 1024     16 GB
2            16      512 x 512       4 GB
3            32      256 x 256       1 GB
4            64      128 x 128       256 MB
5            128     64 x 64         64 MB
6            256     32 x 32         16 MB
========== ========= ============== ===========

In this table, bits/sector denotes the amount of memory required to cut out 27 days worth of 10-minute FFI data at 32 bits per pixel.


Example use
-----------

The `tess-tiler` package provides a fast and memory-efficient way to extract the image data for an entire sector of FFIs, without utilizing more than the memory required by the specific tile. 

.. code-block:: python

    >>> import tess_tiler as tt
    >>> loader = tt.TileLoader(ffi_filenames)
    >>> tpf = loader.get_tpf(zoom=4, x=2, y=3)


How does `tess-tiler` compare to other tools?
---------------------------------------------
Several excellent tools already exist which focus on enabling users to extract or download sub-regions from FFI images via the web, for example:

* ``TessCut`` enables users to download regions up to 100x100 pixels in size centered on an arbitrary coordinate. In contrast, `TessTiler` focuses on extracting larger tiles which cover the entire FFI in a pre-defined tiling scheme.
* ``eleanor`` enables users to download 104-by-148 pixel cutout regions called postcards. *tess-tiler* expands this concept by providing a generic tool to create such postcards at different zoom levels.
