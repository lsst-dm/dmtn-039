..
  Technote content.

  See https://developer.lsst.io/docs/rst_styleguide.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label
     :target: http://target.link/url

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. Add content below. Do not include the document title.

.. note::

   **This technote is not yet published.**

   This note describes work done for DM-7295. It includes instructions for 
   using the LSST Stack to process a set of raw DECam images from ISR through 
   Difference Imaging.
   

.. _Overview:

Overview
========

Ultimately, LSST Alert Production will need to perform all steps of the Level 1
pipeline automatically on each new raw image from the telescope. Details
are in the LSST Data Products Definition Document (DPDD, [jur16]_). Briefly
summarized, the first portion of these steps are:

1. Instrument Signature Removal (ISR), which must include crosstalk correction, bad pixel masking, bias subtraction, and flat field division,
2. Point Source Function (PSF) and background characterization,
3. Photometric and World Coordinate System (WCS) calibration, and
4. Image Differencing with an appropriate template to create Difference Image Analysis (DIA) Source Catalogs.

A number of packages in the LSST Software Stack have been developed to perform
portions of one or more of these tasks, but they are not yet linked together
in an automated way. This Prototype AP Pipeline is the first step toward the goal
of an automatic Level 1 Pipeline that can process a set of raw images through 
each of these steps and verify that the output is what we expect.


.. _Dataset:

Dataset
=======

We use images from the High Cadence Transient Survey (HiTS, [for16]_), a three-year survey 
from 2013--2015 which used the Dark Energy Camera (DECam) on the 4 m Blanco telescope at 
Cerro Tololo Interamerican Observatory (CTIO). The primary science goal of this survey
is to detect and follow up optical transients that have characteristic timescales
from hours to days, with a particular interest in catching early supernovae.

In particular, we select HiTS fields ``Blind15A_38``, ``Blind15A_39``, and ``Blind15A_40``, which
are repeat observations of the same region of sky as ``Blind14A_07``, ``Blind14A_08``, and ``Blind14A_10``
fields, respectively. They are all centered near an RA of 10:15:00 hr and Declination of --04:00:00 deg.
Each field has 34--36 individual visits over about a month, and each DECam image consists of 62 CCDs.
Most of the visits are in the g-band, some are in the r-band, and a few are in z or i.

For the purposes of this prototype pipeline, we consider two g-band visits in ``Blind15A_38`` only.
However, the rest of the visits in fields ``15A_38``--``15A_40`` have been ingested (see details in :ref:`Method <Method>`)
and are ready for the next steps of processing.


.. _Method:

Method
======

To begin, we obtained the dataset by downloading raw images and corresponding master calibration files 
from the `NOAO Science Archive <http://archive.noao.edu/search/query>`_. The HiTS survey data 
may be selected by searching for PI Förster. Additionally, the latest set of defects (bad
pixel masks) were obtained from the ``lsst-dev`` server at ``/datasets/decam/calib/bpmDes/2014-12-05/``.

Next, we ran four main processing steps with the LSST Stack on the data:

1. ``ingestImagesDecam.py``, from the ``obs_decam`` package but drawing heavily on ``pipe_tasks``,
2. ``ingestCalibs.py``, from the ``pipe_tasks`` package,
3. ``processCcd.py``, from the ``pipe_tasks`` package, and
4. ``imageDifference.py``, also from the ``pipe_tasks`` package.

More details coming soon, with references to the ``lsst-dm/decam_hits/hits_ingest.py`` script.




.. _Results:

Results
=======

Some words and some images.



.. [for16] `Förster et al. 2016, ApJ, 832, 155 <http://adsabs.harvard.edu/cgi-bin/nph-data_query?bibcode=2016ApJ...832..155F>`_.
    *The High Cadence Transient Survey (HITS). I. Survey Design and Supernova Shock Breakout Constraints.*

.. [jur16] `Jurić et al. 2016, LSST Document LSE-163 <https://docushare.lsstcorp.org/docushare/dsweb/Get/LSE-163>`_.
    *Large Synoptic Survey Telescope Data Products Definition Document.*

