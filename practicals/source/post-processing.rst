Post processing
===============

This is simply a very basic introduction to some of the more widely used useful tools for viewing, checking, and converting UM input and output data. The tools described below all run on the ARCHER login nodes, and you can run them there, but they also run on the ARCHER post-processors (see http://www.archer.ac.uk/documentation/user-guide/connecting.php#sec-2.1.2). Try logging on to one of the post-processors for these exercises: ``ssh -X espp1``. The post processors can see /home and /work and /nerc.

xconv
-----

**i. View data**

On ARCHER go to the output directory of the global job that you ran previously (the one copied from u-ag761). Run ``xconv`` on the file ending with ``da19880903_00``. This file is an atmosphere start file - this type of file is used to restart the model from the time specified in the file header data.

In the directory above is a file whose name ends in ``.astart``; run a second instance of ``xconv`` on this file. This is the file used by the model to start its run - created by the reconfiguration program in this case.

The ``xconv`` window lists the fields in the file, the dimensions of those fields (upper left panel), the coordinates of the grid underlying the data, the time(s) of the data (upper right panel), some information about the type of file (lower left panel), and general data about the field (lower right panel.)

Both files have the same fields. Double click on a field to reveal its coordinate data. Check the time for this field (select the "t" checkbox in the upper right panel).

Plot both sets of data - click the "Plot Data" button.

View the data - this shows numerical data values and their coordinates and can be helpful for finding spurious data values.

**ii. Convert UM fields data to netCDF**

Select a single-level field (one for which nz=1), choose "Output format" to be "Netcdf", enter an "Output file name", and select "Convert". Information relevant to the file conversion will appear in the lower left panel.

Use ``xconv`` to view the netcdf file just created.

uminfo
------

You can view the header information for the fields in a UM file by using the utility ``uminfo`` - redirect the output to a file or pipe it to ``less``: :: 

  archer$ uminfo <one-of-your-fields-files> | less

The output from this command is best viewed in conjunction with the Unified Model Documentation Paper F3 which explains in depth the various header fields.

Mule
----

Mule consists of a Python API for reading and writing UM files and a set of UM utilities.  This section introduces you to some of the most useful UM utilities.  Full details of Mule can be found on the MOSRS: https://code.metoffice.gov.uk/doc/um/index.html

Before running the mule commands you will need to load the python environment on ARCHER by running: ::

  archer$ module load anaconda

**i. mule-pumf**

This provides another way of seeing header information, but also gives some information about the fields themselves. Its intended use is to aid in quick inspections of files for diagnostic purposes. 

Run ``mule-pumf`` on the start file - here's a couple of examples on one of Ros' files: :: 

 archer$ mule-pumf --print-columns 2 --headers-only \\
                        ag761.astart > ~/mule-pumf-header.out

 archer$ mule-pumf --print-columns 2 ag761.astart > ~/mule-pumf.out

* Can you see what the difference is in the output of these 2 commands?

Take a look at the man page (``mule-pumf -h``) and experiment with some of the other options

**ii. mule-summary**

This utility is used to print out a summary of the lookup headers which describe the fields from a UM file. Its intended use is to aid in quick inspections of files for diagnostic purposes.

Run ``mule-summary`` on the start file again.

**iii. mule-cumf**

This utility is used to compare two UM files and report on any differences found in either the headers or field data. Its intended use is to test results from different UM runs against each other to investigate possible changes. Note, differences in header information can arise even when field data is identical. Try out the following:

* Run ``mule-cumf`` on the two start files referred to above (in the *"View data"* section). You may wish to direct the output to a file.
* Run the same command but with the ``--summary`` option.  This, as the name suggests, prints a much shorter report of the differences.
* Run ``mule-cumf`` on a file and itself.
* View the help page with ``mule-cumf -h`` to find view all the available options. 

um-convpp
---------

We have mentioned in the presentations the PP file format - this is a sequential format (a fields file is random access) still much used in the community. PP data is stored as 32-bit, which provides a significant saving of space, but means that a conversion step is required from a fields file (64-bit). The utility to do this is called ``um-convpp``.  ``um-convpp`` converts directly from 64-bit files produced by the UM to 32-bit PP files.  You must, however, make sure you are using a version 10.4 or greater - you can check that you are using the right one by typing ``which um-convpp``. 

Add the path to ``um-convpp`` to your environment - you can also add this to your ``~/.profile`` so it is available everytime you log in. ::

  archer$ export PATH=$UMDIR/vn10.6/cce/utilities:$PATH

Run ``um-convpp`` on a fieldsfile (E.g ag761a.pc19880901_00) ::

  archer$ cd /home/n02/n02/ros/cylc-run/u-ag761/share/data/History_Data
  archer$ um-convpp ag761a.pc19880901 ag761a.pc19880901.pp

  archer$ ls -l ag761a.pc19880901*
  -rw-r--r-- 1 ros n02 64917504 Mar 15 11:56 ag761a.pc19880901
  -rw-r--r-- 1 ros n02 48581456 Mar 21 10:19 ag761a.pc19880901.pp

Note the reduction in file size. Now use xconv to examine the contents of the PP file.

cfa and cfdump
--------------

There is an increasing use of python in the community and we have, and
continue to develop, python tools to do much of the data processing
previously done using IDL or MATLAB and are working to extend that
functionality. ``cfa`` is a python utility which offers a host of
features - we'll use it to convert UM fields file or PP data to
CF-compliant data in NetCDF format. You first need to set the
environment to run ``cfa`` - if you will be a frequent user, add the
``module load`` and ``module swap`` commands to your .profile. ::

 esPP001$ module load anaconda/2.2.0-python2 cf udunits
 esPP001$ module swap PrgEnv-cray PrgEnv-intel
 esPP001$ cfa -i -o ag761a.pc19880901.pp.nc ag761a.pc19880901.pp
 
Try viewing the NetCDF file with xconv.


``cfdump`` is a tool to view CF fields. It can be run on PP or NetCDF
files, to provide a text representation of the CF fields contained in
the input files. Try it on a PP file and its NetCDF equivalent,
e.g. ::

  archer$ cfdump ag761a.pc19880901.pp | less
  wind_speed field summary
  ------------------------
  Data           : wind_speed(latitude(145), longitude(192)) m s-1
  Cell methods   : time: maximum
  Axes           : time(1) = [1988-09-01 03:00:00] 360_day
                 : altitude(1) = [10.0] m
                 : latitude(145) = [-90.0, ..., 90.0] degrees_north
                 : longitude(192) = [0.0, ..., 358.125] degrees_east
  Aux coords     : model_level_number(altitude(1)) = [8888]

  geopotential_height field summary
  ---------------------------------
  Data           : geopotential_height(latitude(144), longitude(192)) m
  Cell methods   : time: point
  Axes           : time(1) = [1988-09-01 06:00:00] 360_day
                 : air_pressure(1) = [500.0] hPa
                 : latitude(144) = [-89.375, ..., 89.375] degrees_north
                 : longitude(192) = [0.9375, ..., 359.0625] degrees_east
 
CF-python CF-plot
-----------------

Many tools exist for analysing data from NWP and climate models and there are many contributing factors for the proliferation of these analysis utilities, for example, the disparity of data formats used by the authors of the models, and/or the availability of the underlying sofware. There is a strong push towards developing and using python as the underlying language and CF-netCDF as the data format. CMS is home to tools in the CF-netCDF stable - here's an example of the use of these tools to perform some quite complex data manipulations. The user is insulated from virtually all of the details of the methods allowing them to concentrate on scientific analysis rather than programming intricacies.

* Set up the environment and start python - if you will be a frequent
  user, add the ``module load`` and ``module swap`` commands to your
  .profile. ::

   archer$ module load anaconda/2.2.0-python2 cf udunits
   archer$ module swap PrgEnv-cray PrgEnv-intel
   archer$ python
   >>> import cf

We'll be looking at CRU observed precipitation data

* Read in data files ::

  >>> f = cf.read_field('~charles/UM_Training/cru/*.nc')

* Inspect the file contents with different amounts of detail ::

  >>> f
  >>> print f
  >>> f.dump()
  
Note that the three files in the cru directory are aggregated into one
field.

* Average the field with respect to time ::

  >>> f = f.collapse('T: mean')
  >>> print f

Note that the time coordinate is now of length 1.

* Read in another field produced by a GCM, this has a different latitude/longitude grid to regrid the CRU data to ::

  >>> g = cf.read_field('~charles/UM_Training/N96_DJF_precip_means.nc')
  >>> print g

* Regrid the field of observed data (f) to the grid of the model field (g) ::

  >>> f = f.regrids(g, method='bilinear')
  >>> print f

* Subspace the regridded field, f, to a European region ::

  >>> f = f.subspace(X=cf.wi(-10, 40), Y=cf.wi(35, 70))
  >>> print f

Note that the latitude and longitude coordinates are now shorter in length.

* Import the cfplot visualisation library ::

  >>> import cfplot

* Make a default contour plot of the field, f ::

   >>> cfplot.con(f)

* Write out the new field f to disk ::

  >>> cf.write(f, 'cru_precip_european_mean_regridded.nc')

This has just given you a taster of CF-Python & CF-Plot, if you would like to try out some more exercises please take a look at  http://www.met.reading.ac.uk/~swsheaps/scicomp2/index.html
