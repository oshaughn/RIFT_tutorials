
To use RIFT on the IGWN OSG, you need relatively few changes, usually ``--use-osg`` for the top-level pipeline command.  We recommend adjusting the standard tutorial command to add two arguments, as follows:

```
util_RIFT_pseudo_pipe.py --gracedb-id G329473 --approx IMRPhenomD --calibration C01 --make-bw-psds --l-max 2 --choose-data-LI-seglen --add-extrinsic --use-osg --condor-local-nonworker
```
Note that 
* you will also want to set your singularity environment variables, as described below, before running this command.
* OSG IGWN submission requires certain submit nodes.  You will need to launch from an appropriate head node (e.g., ``ldas-osg`` at CIT)



**Special options**

* util_RIFT_pseudo_pipe:
** ``-use-osg``: This sets up OSG use with standard options
** ``--condor-local-nonworker``: This causes non-ILE jobs to run locally (e.g., on the current head node). For very fast jobs, this is much more efficient.
** ``--condor-nogrid-nonworker``: This causes non-ILE jobs to run on a local cluster (i.e., ``DESIRED_SITES=nogrid; flock_local=True``).  Should be better than --condor-local-nonworker


* ILE (usually these are set by the above, but you may need to do this yourself)

* ``create_event_parameter_pipeline_BasicIteration``: Almost no one should be using this program directory.

**Special environment variables**: The following environment variables are used to write into the ``ILE*.sub`` submission files, so these jobs can run on the OSG.
* ``SINGULARITY_BASE_EXE_DIR``: On the OSG, your jobs run in singularity containers.  This directory (usually ``/usr/local/bin``) identifies where in the container RIFT executables are set.
* ``SINGULARITY_RIFT_IMAGE``: On the OSG, your jobs run in singularity containers.  This path identifies which container yo uare running.  Usually ``/cvmfs/singularity.opensciencegrid.org/james-clark/research-projects-rit/rift:latest``
* ``X509_USER_PROXY`` : Usually automatically set for you.   Needed for authenticated CVMFS and gracedb access.
* ``OSG_DESIRED_SITES``: Use to target specific clusters
* ``OSG_UNDESIRED_SITES``: Use to avoid clusters, particularly those where failures or problems consistently occur.  For this and the above, contact us for a list of cluster names.  At time of writing, some cluster names are
```
   CIT
   QB2
   PIC
```
