# water-droplet-hydration

Input files and instructions to run the concerted alchemical simulations for the estimation of the hydration free energies of small to medium sized solute molecules in water droplets using OpenMM described in [Alchemical Transformations for Concerted Hydration Free Energy
Estimation with Explicit Solvation](https://arxiv.org/abs/2005.06504) Currently in press on the Journal of Chemical Physics.

## Contributors

Sheenam Khuttan [ssheenam@gradcenter.cuny.edu](ssheenam@gradcenter.cuny.edu)

Solmaz Azimi [sazimi@gradcenter.cuny.edu](sazimi@gradcenter.cuny.edu)

Joe Wu [jwu1@gradcenter.cuny.edu](jwu1@gradcenter.cuny.edu)

Emilio Gallicchio [egallicchio@brooklyn.cuny.edu](egallicchio@brooklyn.cuny.edu)

## Credits

The repository is maintained by the Gallicchio's laboratory at Department of Chemistry of Brooklyn College of CUNY [(compmolbiophysbc.org)](compmolbiophysbc.org). The work is supported in part from a grant from the National Science Foundation (CAREER 1750511).


## Documentation and Tutorials

The project walks you over the steps to setup and run alchemical absolute binding free energy calculations for a series of solute complexes with water droplet (TIP3P model) using the Single Decoupling (SDM) OpenMM's plugin.

## Gathering Software Tools

### OpenMM

We recommend installing OpenMM from [source](https://github.com/openmm/openmm) since most of the steps required are the same as for building the SDM-related plugins. However, binary installations of OpenMM should probably also work. Detailed building instructions for OpenMM are [here](http://docs.openmm.org/latest/userguide/library.html#compiling-openmm-from-source-code). SDM requires an OpenCL platform with GPUs from NVIDIA (CUDA) or AMD, which we assume are in place. We developed a [docker image](https://hub.docker.com/repository/docker/egallicchio/centos610-openmmbuilder) with all of the tools to build OpenMM. 


### Desmond File Reader for OpenMM

SDM uses Desmond DMS-formatted files. OpenMM includes a python library to load molecular files in DMS Desmond format; however, the version of the Desmond file reader required for SDM is not yet included in the latest OpenMM sources. To patch the 7.3.1 OpenMM installation above with the latest DMS file reader, do for example:

```
cd $HOME/devel
wget https://raw.githubusercontent.com/egallicc/openmm/master/wrappers/python/simtk/openmm/app/desmonddmsfile.py
cp desmonddmsfile.py $HOME/devel/openmm-7.3.1/wrappers/python/simtk/openmm/app/
```

Then rebuild OpenMM from the sources in ```$HOME/devel/openmm-7.3.1```.

The ```sqlitebrowser``` application is very useful to inspect DMS files.

### ASyncRE for OpenMM

ASyncRE is a package written in python to perform replica exchange (RE) simulations in asynchronous mode across a set of GPUs. To install it into a conda environment, do:

```
conda install numpy configobj
cd $HOME/devel
git clone https://github.com/egallicc/async_re-openmm.git
cd async_re-openmm
python setup.py install
```
### VMD

[VMD](http://www.ks.uiuc.edu/Research/vmd/) is an indispensable tool for visualization and analysis of trajectories. Also, we use the catdcd tool distributed with VMD to parse and concatenate .dcd trajectory files. For this tutorial we will assume that VMD is installed in /usr/local.


## Run the ASyncRE simulations

Complex directories of water-droplet with solutes (ethanol, 1-naphthol, diphenyltoluene and alanine dipeptide) with both linear and logistic potential settings are available in the repository. The files have been produced by setting up the simulation parameters for each complex, defining the alchemical schedule and running the workflow for setting up the receptor, and then each complex in turn.

Go to the simulation directories of each complex and launch the ASyncRE simulations. For example, assuming ASyncRE is installed under ```$HOME/devel/async_re-openmm```:

To run simulation of the ethanol solute with water droplet under linear alchemical schedule, go to directory **EtOH-lin** *(**-lin** complex directories specify **linear alchemical schedule**, **-log** complex directories specify **logistic alchemical schedule with softplus parameters**.)*

Set the `nodefile` to point to a set of GPUs on the local machine. For example:

```
locahost,0:0,1,OpenCL,,/tmp
locahost,0:1,1,OpenCL,,/tmp
locahost,0:2,1,OpenCL,,/tmp
locahost,0:3,1,OpenCL,,/tmp
```

for a compute server with 4 GPUs when the OpenCL platform is the first platform (0).

```
./runopenmm $HOME/devel/async_re-openmm/bedamtemptwd_async_re.py water-droplet-ethanol_asyncre.cntl
```

Adjust the settings in the ```runopenmm``` script to reflect your environment.

Each RE simulation is set to run for 480 mins (8 hours). You can change the duration of the simulation run by changing the ```WALL_TIME``` in the ```*_asyncre.cntl``` file present in each complex directory.
Monitor the progress of each ASyncRE simulation by inspecting the ```*_stat.txt``` file. For example:

```
cd $HOME/water-droplet/complexes/EtOH-lin
cat water-droplet-ethanol_stat.txt
```
Each replica runs in a separate subdirectory named ```r0```, ```r1```, etc. Each cycle generates ```.out```, ```.dms```, ```.log```, ```.pdb```, and ```.dcd``` files. For example the binding energy samples for the first cycle of replica 2 are stored in the file ```r2/water-droplet-ethanol_1.out``` . The .dms files are used to start the following cycle. The .pdb and .dcd files are used for trajectory visualization.
 
The ASyncRE process can be killed at any time with ```^C``` and optionally restarted. However, replicas currently running on remote machines are likely to keep running and may have to be killed before ASyncRE can be restarted. To start from scratch (that is from the first cycle) remove the replicas directories by doing ```rm -r r? r??```.

## Output

Once the simulation has started, replica directories pointing at each lambda value in the alchemical schedule are produced. 
In each replica directory, a  ```*.out``` is created producing columns of values in the order: temperature, lambda, lambda1, lambda2, alpha, u0, w0, total potential energy, binding energy.

This output file can then be used to analyze the average value of different parameters over the course of simulation.

Additionally, to visualize the trajectories of the solute-complex binding, ```*.dcd``` files are produced which can be used along with the ```*.pdb``` file in the complex directory with VMD.

```
cd $HOME/water-droplet/complexes/EtOH-lin
vmd -f water-droplet-ethanol_6.pdb water-droplet-ethanol.dcd
```










