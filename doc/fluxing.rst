=======
Fluxing
=======

Overview
========
Fluxing is done after the main run of PypeIt.

It is a two step process of generating a `Sensitivity Function`_
and then `Applying the Sensitivity Function`_.
We describe each in turn.



Sensitivity Function
====================

The sensitivity function is generated from the
:doc:`out_spec1D` file of a processed standard star.

PypeIt uses an archived fluxed spectrum from either
the `CALSPEC calibration database <http://stsci.edu/hst/observatory/crds/calspec.html>`_
or one of the files we have grabbed from
`ESO <https://www.eso.org/sci/observing/tools/standards/spectra/stanlis.html>`_.
If you observed something else, see `Adding a Standard Star`_.


The sensitivity function is generated by dividing the standard
star's flux by the
standard star's counts per second. This is then multiplied to the
science object's counts per second to yield a fluxed science
spectrum.

The sensitivity function is written to disk as a FITS file.
This function converts the extracted count spectrum
to f_lambda units of 1e-17 erg/s/cm^2/Ang.

pypeit_sensfunc
---------------

The process is mediated by the *pypeit_sensfunc* script.
Its usage (*pypeit_sensfunc -h*) describes its functionality.
Here is a typical call::

    pypeit_sensfunc spec1dfile -o Keck_LRISr_600_7500_sens.fits

This analyzes the standard star spectrum in *spec1dfile* and writes
the sensitivity file to *Keck_LRISr_600_7500_sens.fits*.

Here are the common options used:

--multi
+++++++

For some instruments (e.g. *keck_deimos*, *gemini_gmos*), the spectrum spans
across multiple detectors.  You can have the sensitivity function
handle this by using the --multi option, e.g.::

    pypeit_sensfunc --multi 3,7

--debug
+++++++

Throws a number of plots to the screen

--algorithm
+++++++++++

The algorithm options are:
 - UVIS = Should be used for data with lambda < 7000A.
   No detailed model of telluric absorption but corrects for atmospheric extinction.
 - IR   = Should be used for data with lambbda > 7000A.
   Peforms joint fit for sensitivity function and telluric absorption using HITRAN models.

--sens
++++++

Provide a file to guide the process.  Do this if your changes to
the defaults are not accommodated by the script inputs.

IR without a Standard
---------------------

If you wish to generate a sensitivity function on a standard
star that is not part of the PypeIt database and are working
in the IR, you can feed the stellar parameters.  Here is an
example::

    [sensfunc]
       algorithm = IR
       star_mag = 12.1
       star_type = A0

Then run on the spec1d file as you would otherwise.
For an A0 star, we use the Vega spectrum.  Otherwise,
we use the Kurucz93 stellar SED.

Alternative see `Adding a Standard Star`_.

Applying the Sensitivity Function
=================================

Once you have generated a `Sensitivity Function`_, you may apply
it to one or more :doc:`out_spec1D` files.
The files are modified in place, filling the OPT_FLAM, BOX_FLAM, etc.
entries, as described in :doc:`specobj`.

Flux File
---------

To flux one or more spec1d files, generate a flux_file that is has the
following format::

    flux read
       spec1dfile1 sensfile
       spec1dfile2
          ...
          ...
    flux end

    OR

    flux read
       spec1dfile1 sensfile1
       spec1dfile2 sensfile2
       spec1dfile3 sensfile3
          ...
    flux end

Here is an actual example::

    flux read
      spec1d_UnknownFRBHostY_vlt_fors2_2018Dec05T020241.687.fits VLT_FORS2_sens.fits
      spec1d_UnknownFRBHostY_vlt_fors2_2018Dec05T021815.356.fits
      spec1d_UnknownFRBHostY_vlt_fors2_2018Dec05T023349.816.fits
    flux end

If one wishes to modify the :ref:`pypeit_par:FluxCalibratePar Keywords`,
add a Parameter block at the top of the file, e.g.::

    [fluxcalib]
       extrap_sens = True

    flux read
      spec1d_FORS2.2019-07-12T08:11:41.539-FRB190611Host_FORS2_2019Jul12T081141.539.fits VLT_FORS2_300I_sens.fits
      spec1d_FORS2.2019-07-12T08:34:55.904-FRB190611Host_FORS2_2019Jul12T083455.904.fits
    flux end


pypeit_flux_calib
-----------------

Fluxing is performed with the *pypeit_flux_calib* script.
Use *pypeit_flux_calib -h* to see its full usage.  Here is a
typical call::

    pypeit_flux_calib flux_file.txt

Again, the :doc:`out_spec1D` files are modified in place.
See :ref:`pypeit-1dspec` for details on how to view them.

FluxSpec Class
==============

The guts of the flux algorithms are guided by the
:class:`pypeit.fluxcalibrate.FluxCalibrate`.
class.

Troubleshooting
===============

Problem with bspline knot
-------------------------

Adding a Standard Star
======================

If your star is not in the repository you can add in a new
solution if it is in the
`ESO database <https://www.eso.org/sci/observing/tools/standards/spectra/stanlis.html>`_.

You will need to place their .dat file in pypeit/data/standards/esofil/
and then edit the *esofil_info.txt* file in their accordingly.
Extra kudos if you submit this as a PR for others benefit.

If your standard star is even more non-traditional, contact
the developers.
