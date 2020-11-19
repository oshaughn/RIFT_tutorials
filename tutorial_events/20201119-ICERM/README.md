# ICERM 2020-11 tutorial


## Tutorial 0: Principles of iterative fitting

Open the file [Demo-RIFT_Principles](Demo-RIFT_Principles.ipynb).  If you don't have a jupyter notebook viewer on your own machine, 
you can download and re-upload this to ``colab.research.google.com``.  This notebook only requires standard python packages (numpy,scipy, sklearn, matplotlib)


## Tutorial 1: RIFT in action

This tutorial generates a simple binary black hole gravitational wave signal in a 2-detector network,
then recovers its parameters under the assumption of known gaussian noise in these two detectors.
The iterative grid is very small.

Unlike the python example above, where everything was done inline, each step of the overall calculation is coordinated by a scheduler (condor).  
When we set up our workflow, we decide how many iterations we will perform, and specify the details of how each sub-step of the calculation is performed.
Each step is performed by some command-line executable.

Also unlike the calculation above, this tutorial uses  ``ILE`` (integrate_likelihood_extrinsic_batchmode), a routine which calculates a (marginal) likelihood for each candidate set of binary parameters (here, m1 and m2) by comparing a proposed GW signal to our synthetic data.    ``ILE`` replaces the ``lnL_true`` function we used in the toy model calculation above.

Similar to the  calculation above, this tutorial uses ``CIP`` (construct_intrinsic_posterior_GenericCoordintaes.py) to construct an approximation to the likelihood based on our training data (from all ILE evaluations in the past); and to construct an approximate posterior distribution based on this approximate likelihood.  Also similar to the toy model, a program ``puffball`` will supplement our posterior with additional points jittered by random errors, to insure wide parameter coverage.  

**Step-by-step process**
* Step 0: Install prerequisite: Docker (docker.com).  [Note: if you have /cvmfs and can use either pre-built containers or conda environments, please see [INSTALL.md](https://github.com/oshaughn/research-projects-RIT/blob/master/INSTALL.md) instead, and skip to step 3]
* Step 1: Start the container used for this tutorial, available on docker hub as ``jclarkastro/rift-demo-cuda92:latest`` with [this dockerfile](https://git.ligo.org/james-clark/benchmarking/-/blob/master/RIFT/full-demo/rift-demo-cuda92.Dockerfile)
```
docker run --detach --gpus all --cpuset-cpus 0-3 --name=rift-demo -v ${HOME}:${HOME} jclarkastro/rift-demo-cuda92:latest   # start the container; this should run ./start-condor.sh
docker exec -it --user albert rift-demo bash  # join the container as user 'albert', not root
```
If you don't have GPUs available, please don't use the ``--gpus`` option.  Please adjust the ``/home:/home`` option to be appropriate for your filesystem.

You can now monitor the internal condor queue system with ``condor_q`` and ``condor_status``.  As time permits, please look at any [condor tutorial](https://research.cs.wisc.edu/htcondor/tutorials/intl-grid-school-3/submit_first.html) to understand how that queuing system works.

When you are **done**, you should close the container with 
```
docker stop rift-demo
docker rm rift-demo
```
* Step 2 (optional): Confirm your installation works: Follow the **Confirming your installation works** steps at the [RIFT getting started](https://github.com/oshaughn/research-projects-RIT/blob/master/GETTING_STARTED.md) tutorial
* Step 3: Set up and launch a simple analysis
```
# Download the analysis
#   - should already be preent in /rift-demos
# EITHER DO THIS
git  clone https://github.com/oshaughn/ILE-GPU-Paper.git
cd ILE-GPU-Paper/demos/
# OR DO THIS
cd /rift-demos
```

```
# Set up the analysis
make test_workflow_nobatch   # IF YOU HAVE **NO** GPUS
#make test_workflow_batch_gpu_lowlatency   # IF YOU HAVE GPUS
cd test_workflow_nobatch; ls
# Launch the analysis
condor_submit_dag marginalize_intrinsic_parameters_BasicIterationWorkflow.dag
# Monitor the progress of the jobs
watch -n 10 condor_q
```
Again, the [RIFT getting started](https://github.com/oshaughn/research-projects-RIT/blob/master/GETTING_STARTED.md) page describes the files being created.

**Troubleshooting**
* *My jobs don't start?* Depending on your machine, you may not have a lot of memory allocated per slot.  You may also not have any GPUs.  The example above assumes 4Gb of RAM per slot and a GPU per slot in ILE.sub.  You can change ``request_GPUs=1`` to ``request_GPUs=0`` and ``request_memory=4096`` to ``request_memory=1000`` to adjust the requirements accordingly.
* *My job doesn't produce output?* : You can add the following two lines to ``ILE.sub``
```
stream_output=True
stream_error=True
```

* Step 4: Plot some results
```
 plot_posterior_corner.py --posterior-file posterior-samples-5.dat --parameter mc --parameter eta --plot-1d-extra --ci-list  '[0.9]'  --composite-file all.net --quantiles None  --use-legend 
```


## Tutorial 2: Production-quality analysis

The procedure described in tutorial 1 is exhausting for real analysis: we need to chooise a grid, prior range, iteration structure, and other settings (e.g., the frequency of puffball).  For production-quality analysis, we have encoded best practices into ``util_RIFT_pseudo_pipe.py``.   If you have used LALInference, you can immediately re-use your configuration to analyze with our script

```
util_RIFT_pseudo_pipe.py --use-ini my.ini --use-coinc coinc.xml --use-osg --cip-explode-jobs 3
```

```
util_RIFT_pseudo_pipe.py --gracedb-id G329473 --approx IMRPhenomD --calibration C01 --make-bw-psds --l-max 2 --choose-data-LI-seglen --add-extrinsic
```




# Bonus tutorials

The docker image ``jclarkastro/rift-demo-cuda92:latest`` comes  pre-installed with 
* frames and a PSD for a real analysis used for benchmarking
* test jobs (
