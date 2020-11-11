#ICERM 2020-11 tutorial



* Step 0: Install prerequisite: Docker (docker.com)
* Step 1: Start the container used for this tutorial
``
docker run --detach --gpus all -v /home:/home  --name=rift-demo jclarkastro/rift-demo-cuda92:latest
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
