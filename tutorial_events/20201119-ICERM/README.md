#ICERM 2020-11 tutorial


==Tutorial 0: Principles of iterative fitting==

Open the file [Demo-RIFT_Principles](Demo-RIFT_Principles.ipynb).  If you don't have a jupyter notebook viewer on your own machine, 
you can download and re-upload this to ``colab.research.google.com``.  This notebook only requires standard python packages (numpy,scipy, sklearn, matplotlib)


==Tutorial 1: RIFT in action==

This tutorial generates a simple binary black hole gravitational wave signal in a 2-detector network,
then recovers its parameters under the assumption of known gaussian noise in these two detectors.
The iterative grid is very small.

Unlike the python example above, where everything was done inline, each step of the overall calculation is coordinated by a scheduler (condor).  
When we set up our workflow, we decide how many iterations we will perform, and specify the details of how each sub-step of the calculation is performed.
Each step is performed by some command-line executable.

Also unlike the calculation above, this step involves ``ILE``, a routine which calculates a (marginal) likelihood for each candidate set of binary parameters (here, m1 and m2) by comparing a proposed GW signal to our synthetic data.    ``ILE`` replaces the ``lnL_true`` function we used in the toy model calculation above.


**Step-by-step process**
* Step 0: Install prerequisite: Docker (docker.com)
* Step 1: Start the container used for this tutorial
``
 docker run -i -t   -v /home:/home  --name=rift-demo -v `pwd`/home:/home jclarkastro/rift-demo-cuda92:latest /bin/bash
 cd /;  nohup ./start.sh &  # start condor
 ln -sf /usr/bin/python3 /usr/bin/python  # force python3
#docker run --detach --gpus all -v /home:/home  --name=rift-demo jclarkastro/rift-demo-cuda92:latest
``
If you don't have GPUs available, please don't use the ``--gpus`` option.  Please adjust the ``/home:/home`` option to be appropriate for your filesystem
* Step 2: Confirm your installation works: Follow the **Confirming your installation works** steps at the [RIFT getting started](https://github.com/oshaughn/research-projects-RIT/blob/master/GETTING_STARTED.md) tutorial
* Step 3: Set up and launch a simple analysis
```
# Download the analysis
git  clone https://github.com/oshaughn/ILE-GPU-Paper.git
cd ILE-GPU-Paper/demos/
# Set up the analysis
make test_workflow_batch_gpu_lowlatency
cd test_workflow_batch_gpu_lowlatency; ls
# Launch the analysis
condor_submit_dag marginalize_intrinsic_parameters_BasicIterationWorkflow.dag
# Monitor the progress of the jobs
watch -n 10 condor_q
```
Again, the [RIFT getting started](https://github.com/oshaughn/research-projects-RIT/blob/master/GETTING_STARTED.md) page describes the files being created.
* Step 4: Plot some results
```
 plot_posterior_corner.py --posterior-file posterior-samples-5.dat --parameter mc --parameter eta --plot-1d-extra --ci-list  '[0.9]'  --composite-file all.net --quantiles None  --use-legend 
```


==Tutorial 2: Production-quality analysis==
