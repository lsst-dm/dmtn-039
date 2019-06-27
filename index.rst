:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. Add content below. Do not include the document title.

.. warning::

   This document describes an early prototype of the LSST Alert Generation system.
   As of mid-2019, much of the material presented here is obsolete.
   Please refer to the `Science Pipelines documentation`__ for current information.

   __ https://pipelines.lsst.io


.. _Overview:

Overview
========

Ultimately, LSST Alert Production (AP) will need to perform all steps of the Level 1
pipeline automatically on each new raw image from the telescope. Details
are in the LSST Data Management Science Pipelines Design  (LDM-151 :cite:`LDM-151`) and
the LSST Data Products Definition Document (DPDD :cite:`LSE-163`). Briefly
summarized below and in :ref:`Figure 1<fig-nightly-proc>`, the first portion of these steps are:

1. Instrument Signature Removal (ISR), which includes crosstalk and linearity corrections, bad pixel masking, bias subtraction, and flat field division,
2. Point Source Function (PSF) and background characterization, which includes source detection and measurement,
3. Photometric and World Coordinate System (WCS) calibration, and
4. Image Differencing with an appropriate template (which may need constructing) to create Difference Image Analysis (DIA) source catalogs.

.. figure:: /_static/nightly_processing_1.png
   :name: fig-nightly-proc
   :alt: First portion of the LDM-151 Nightly Overview flowchart, from ISR to Image Differencing

   The first portion of the Nightly Processing Pipeline overview, from Figure 1 of
   :cite:`LDM-151`. This encompasses all of "Single Frame Processing" and part of "Alert Detection."

A number of packages in the LSST Software Stack have been developed to perform
portions of one or more of these four tasks, but they are not yet linked together
in an automated way. This Prototype AP Pipeline is the first step toward the goal
of an automatic Level 1 Pipeline that can process a set of raw images through
each of these steps and verify that the output is what we expect.

To achieve this, there are two main python modules under development:
`ap_pipe <https://github.com/lsst-dm/ap_pipe>`_ and `ap_verify <https://github.com/lsst-dm/ap_verify>`_.
The former, described here, is responsible for running the Prototype Pipeline. The latter
uses ``ap_pipe`` to verify the output.


.. _Dataset:

Dataset
=======

We use images from the High Cadence Transient Survey (HiTS) :cite:`for16`, a three-year survey
from 2013--2015 which used the `Dark Energy Camera (DECam) <http://www.ctio.noao.edu/noao/content/DECam-What>`_
on the 4 m Blanco telescope at Cerro Tololo Interamerican Observatory (CTIO).
The primary science goal of this survey is to detect and follow up optical transients
that have characteristic timescales from hours to days, with a particular interest in catching early supernovae.
The distribution of the HiTS survey fields in the sky is shown in :ref:`Figure 2<fig-hits>`.

.. figure:: /_static/forster_fig4.png
   :name: fig-hits
   :alt: Distribution of the HiTS survey fields in the sky, color-coded by year

   Spatial distribution of the fields observed in the 2013--2015 HiTS campaign (Figure 4 from :cite:`for16`).
   The fields selected for this dataset are three of the pink ones, which indicate they were visited in both 2014 and 2015.
   Specifically, they are the pair of partially-overlapping pink fields near the bottom center of the 2014 and 2015
   field region and the upper-right-most pink field.

We select HiTS fields ``Blind15A_26``, ``Blind15A_40``, and ``Blind15A_42``, which
are repeat observations of the same region of sky as ``Blind14A_04``, ``Blind14A_10``, and ``Blind14A_09``
fields, respectively. Each field has 34--36 individual visits over about a month, and each DECam image
consists of 62 CCDs (however, two of these are non-functional).
Most of the visits are in the *g*-band, some are in the *r*-band, and a few are in *i*.

All of these data are in the git-lfs repo `ap_verify_hits2015 <https://github.com/lsst/ap_verify_hits2015>`_
in raw format, along with the corresponding DECam Master Calibration files and camera defect files.
The data also exist on the ``lsst-dev`` server at ``/datasets/decam/_internal/raw/hits`` with the
calibration files at ``/dataset/decam/_internal/calib/cpHits`` and the camera defects at
``/dataset/decam/_internal/calib/bpmDes``. The raw images and calibration files were originally obtained
from the `NOAO Science Archive <http://archive.noao.edu/search/query>`_ by searching for PI Förster.


.. _Tutorial:

Tutorial
========

This tutorial walks a user through using ``ap_pipe`` to run four main processing steps
with the LSST Stack on a portion of the :ref:`Dataset <Dataset>` described above:

1. ``ingestImagesDecam.py``, from the ``obs_decam`` package but drawing heavily on ``pipe_tasks``,
2. ``ingestCalibs.py``, from the ``pipe_tasks`` package,
3. ``processCcd.py``, from the ``pipe_tasks`` package, and
4. ``imageDifference.py``, also from the ``pipe_tasks`` package.

The prerequisites for running ``ap_pipe`` are:

- The `LSST Science Pipelines <https://pipelines.lsst.io/install/lsstsw.html>`_
  stack with the ``lsst_apps`` packages (a subset of ``lsst_distrib``)
- The ``obs_decam`` and ``ap_pipe`` packages
- A clone of the ``ap_verify_hits2015`` dataset

You may set all of this up as follows:

.. prompt:: bash

   setup lsst_apps
   # If using the lsst-dev server, lsst_apps may be setup using a centrally
   # available weekly build, but the following packages are not included
   git clone https://github.com/lsst/obs_decam.git
   git clone https://github.com/lsst-dm/ap_pipe.git
   setup -k -r obs_decam
   setup -k -r ap_pipe
   git clone https://github.com/lsst/ap_verify_hits2015.git

Once you are ready, run ``ap_pipe`` from the command line. You must point to the dataset
with the ``-d`` flag, a desired output location on disk with the ``-o`` flag, and provide
a valid visit and ccdnum dataId string with the ``-i`` flag.

.. prompt:: bash

   python ap_pipe/bin.src/ap_pipe.py -d ap_verify_hits2015/ -o output_dir -i "visit=410985 ccdnum=25"

.. note::

    At present (`DM-11390 <https://jira.lsstcorp.org/browse/DM-11390>`_), the template used for
    difference imaging is hard-wired to visit 410929 and ccdnum 25.
    This is a single CCD only of one of the ``Blind15A_40`` visits. If you would like to use
    a different template, you must manually set this in the source code
    (``ap_pipe/python/lsst/ap/pipe/ap_pipe.py``, in the function ``runPipelineAlone``).
    This functionality will be improved when we switch from using a visit as a template to using coadds
    by default (see `DM-11422 <https://jira.lsstcorp.org/browse/DM-11422>`_).

This will run all four steps of the pipeline in order. Each step will create a new subdirectory
in ``output_dir``: one for ingested images, one for ingested calibration products, one for
processed images, and one for difference imaging.


.. _Results:

Results
=======

The difference image and DIA Source catalog for a test image are available
on the ``lsst-dev`` server at ``/project/mrawls/prototype_ap/zomg7/diffim/deepDiff/v410985/``.
A screenshot of the calibrated exposure (calexp) and difference image is shown
in :ref:`Figure 3<fig-diffim>`.

.. figure:: /_static/diffim_v410985.png
   :name: fig-diffim
   :alt: Calibrated exposure and difference image for DECam visit 410985, CCD 25

   Calibrated science exposure (top) and difference image (bottom) for CCD 25 of HiTS field ``Blind15A_40``.
   This example uses visit 410929 as the template image and visit 410985 as the science image.

Future work will use coadded 2014 visits of the three HiTS fields described in Dataset
as templates and the corresponding 2015 visits as science.
This pipeline is a core component of the
`AP Minimum Viable System <https://confluence.lsstcorp.org/display/~ebellm/AP+Minimum+Viable+System>`_
(note this link is private to LSST team members only).
The goal is to run and verify each component of LSST image processing as we build a
fully functional ``ap_verify`` system.
The ``ap_pipe`` pipeline will also be available to users who wish to process any DECam dataset independently.

.. _References:

References
==========

.. bibliography:: local.bib
   :encoding: latex+latin
   :style: lsst_aa

