==========
Quickstart
==========

Loading data
============

.. testsetup:: *
    >>> import sys
    >>> reload(sys)
    >>> sys.setdefaultencoding('utf8')

To work with weather satellite data you must create a :class:`~satpy.scene.Scene` object. In order for SatPy to
get access to the data it must be told what files to read and what :ref:`SatPy Reader <reader_table>` should read them::

    >>> from satpy import Scene
    >>> from glob import glob
    >>> filenames = glob("/home/a001673/data/satellite/Meteosat-10/seviri/lvl1.5/2015/04/20/HRIT/*201504201000*")
    >>> global_scene = Scene(reader="hrit_msg", filenames=filenames)

To load data from the files use the :meth:`Scene.load <satpy.scene.Scene.load>` method::

    >>> global_scene.load([0.6, 0.8, 10.8])
    >>> print(global_scene)

    seviri/IR_108:
            area: On-the-fly area
            start_time: 2015-04-20 10:00:00
            units: K
            wavelength_range: (9.8, 10.8, 11.8) μm
            shape: (3712, 3712)
    seviri/VIS006:
            area: On-the-fly area
            start_time: 2015-04-20 10:00:00
            units: %
            wavelength_range: (0.56, 0.635, 0.71) μm
            shape: (3712, 3712)
    seviri/VIS008:
            area: On-the-fly area
            start_time: 2015-04-20 10:00:00
            units: %
            wavelength_range: (0.74, 0.81, 0.88) μm
            shape: (3712, 3712)

SatPy allows loading file data by wavelengths in micrometers (shown above) or by channel name::

    >>> global_scene.load(["VIS006", "VIS008", "IR_108"])

To have a look at the available channels for loading from your :class:`~satpy.scene.Scene` object use the
:meth:`available_datasets <satpy.scene.Scene.available_dataset_names>` method::

    >>> global_scene.available_dataset_names()

    ['HRV',
     'IR_108',
     'IR_120',
     'VIS006',
     'WV_062',
     'IR_039',
     'IR_134',
     'IR_097',
     'IR_087',
     'VIS008',
     'IR_016',
     'WV_073']


To access the loaded data use the wavelength or name::

    >>> print(global_scene[0.6])

To visualize loaded data in a pop-up window::

    >>> global_scene.show(0.6)

To make combine datasets and make a new dataset::

    >>> global_scene["ndvi"] = (global_scene[0.8] - global_scene[0.6]) / (global_scene[0.8] + global_scene[0.6])
    >>> global_scene.show("ndvi")

For more information on loading datasets by resolution, calibration, or other
advanced loading methods see the :doc:`readers` documentation.

Generating composites
=====================

SatPy comes with many composite recipes built-in and makes them loadable like any other dataset::

    >>> global_scene.load(['overview'])

To get a list of all available composites for the current scene::

    >>> global_scene.available_composite_names()

    ['overview_sun',
     'airmass',
     'natural',
     'night_fog',
     'overview',
     'green_snow',
     'dust',
     'fog',
     'natural_sun',
     'cloudtop',
     'convection',
     'ash']

Loading composites will load all necessary dependencies to make that composite and unload them after the composite
has been generated.

.. note::

    Some composite require datasets to be at the same resolution or shape. When this is the case the Scene object must
    be resampled before the composite can be generated (see below).

Resampling
==========

.. todo::

   Explain where and how to define new areas

In certain cases it may be necessary to resample datasets whether they come
from a file or are generated composites. Resampling is useful for mapping data
to a uniform grid, limiting input data to an area of interest, changing from
one projection to another, or for preparing datasets to be combined in a
composite (see above). For more details on resampling, different resampling
algorithms, and creating your own area of interest see the
:doc:`resample` documentation. To resample a SatPy Scene::

    >>> local_scene = global_scene.resample("eurol")

This creates a copy of the original ``global_scene`` with all loaded datasets
resampled to the built-in "eurol" area. Any composites that were requested,
but could not be generated are automatically generated after resampling. The
new ``local_scene`` can now be used like the original ``global_scene`` for
working with datasets, saving them to disk or showing them on screen::

    >>> local_scene.show('overview')
    >>> local_scene.save_dataset('overview', './local_overview.tif')

Saving to disk
==============

To save all loaded datasets to disk as geotiff images::

    >>> global_scene.save_datasets()

To save all loaded datasets to disk as PNG images::

    >>> global_scene.save_datasets(writer='simple_image')

Or to save an individual dataset::

    >>> global_scene.save_dataset('VIS006', 'my_nice_image.png')

Datasets are automatically scaled or "enhanced" to be compatible with the
output format and to provide the best looking image. For more information
on saving datasets and customizing enhancements see the documentation on
:doc:`writers`.
