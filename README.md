# water-droplet-hydration
Our paper discusses methods for concerted estimation of hydration free energies of small to medium sized solute molecueles using softplus potential. [arXiv:2005.06504](https://arxiv.org/abs/2005.06504)
We present a repository of simulation files for water-droplet and solute complexes with instructions for alchemical binding free energy calculations with OpenMM.

## Contributers

Emilio Gallicchio [egallicchio@brooklyn.cuny.edu](egallicchio@brooklyn.cuny.edu)

Sheenam Khuttan [ssheenam@gradcenter.cuny.edu](ssheenam@gradcenter.cuny.edu)

Solmaz Azimi [sazimi@gradcenter.cuny.edu](sazimi@gradcenter.cuny.edu)

Joe Wu [jwu1@gradcenter.cuny.edu](jwu1@gradcenter.cuny.edu)

## Credits

This software is maintained by the Gallicchio's laboratory at Department of Chemistry of Brooklyn College of CUNY [(compmolbiophysbc.org)](compmolbiophysbc.org). Development and maintenance of this software is supported in part from a grant from the National Science Foundation (CAREER 1750511).


## Documentation and Tutorials

The project walks you over the steps to setup and run alchemical absolute binding free energy calculations for a series of  solute complexes with water droplet (TIP3P model) using the Single Decoupling (SDM) OpenMM's plugin.

## Gathering Software Tools

### OpenMM

We recommend installing OpenMM from [source](https://github.com/openmm/openmm) since most of the steps required are the same as for building the SDM-related plugins. However binary installations of OpenMM should probably also work. Detailed building instructions for OpenMM are [here](http://docs.openmm.org/latest/userguide/library.html#compiling-openmm-from-source-code). SDM requires an OpenCL platform with GPUs from NVIDIA (CUDA) or AMD, which we assume are in place.

These are the steps we used to build OpenMM 7.3.1 on an Ubuntu 16.04 system:

```
mkdir $HOME/devel
cd $HOME/devel
wget https://github.com/pandegroup/openmm/archive/7.3.1.tar.gz
tar zxvf 7.3.1.tar.gz
conda install cmake=3.6.3 swig numpy
conda install -c conda-forge doxygen
mkdir build_openmm
cd build_openmm
ccmake -i ../openmm-7.3.1
```

Hit ```c``` (configure) until all variables are correctly set. Set ```CMAKE_INSTALL_PREFIX``` to point to the openmm installation directory. Here we assume ```$HOME/local/openmm-7.3.1```. If an OpenCL platform is detected (such as from a NVIDIA CUDA installation) it will be enabled automatically. For the present purposes the CUDA platform is optional. Hit ```g``` to generate the makefiles and ```q``` to exit ```ccmake```, then:

```
make install
make PythonInstall
```

The OpenMM libraries and header files will be installed in ```$HOME/local/openmm-7.3.1```

### Desmond File Reader for OpenMM

SDM uses Desmond DMS-formatted files. OpenMM includes a python library to load molecular files in DMS Desmond format however, the version of the Desmond file reader required for SDM is not yet included in the latest OpenMM sources. To patch the 7.3.1 OpenMM installation above with the latest DMS file reader do the following:

```
cd $HOME/devel
wget https://raw.githubusercontent.com/egallicc/openmm/master/wrappers/python/simtk/openmm/app/desmonddmsfile.py
cp desmonddmsfile.py $HOME/devel/openmm-7.3.1/wrappers/python/simtk/openmm/app/
cd $HOME/devel/build_openmm
make install
make PythonInstall
```
 
The ```sqlitebrowser``` application is very useful to inspect DMS files.

### ASyncRE for OpenMM

ASyncRE is a package written in python to perform replica exchange simulations in asynchronous mode across a wide range of computational devices and grids. The specific version used by SDM distributes OpenMM jobs across GPU compute servers through passwordless ssh. To install it do:

```
conda install numpy configobj paramiko
cd $HOME/devel
git clone https://github.com/egallicc/async_re-openmm.git
cd async_re-openmm
python setup.py install
```
Here is a [guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-1604) to set up password-less ```ssh``` across a farm of computers. ASyncRE does not require that the compute servers share a common user space and a shared filesystem.

## Run the ASyncRE simulations

Complex directories of water-droplet with solutes (Ethanol, 1-napthol, Diphenyltoluene and Alanine-dipeptide) with both linear and logistic potential settings are available in the repository. The files have been produced by setting up the simulation parameters for each complex, defining the alchemical schedule and running the workflow for setting up the receptor, and then each complex in turn.

Go to the simulation directories of each complex and launch the ASyncRE simulations. For example, assuming ASyncRE is installed under ```$HOME/devel/async_re-openmm```:

To run simualtion of Ethanol solute with water droplet under linear alchemical schedule, go to directory **EtOH-lin** *(**-lin** complex directories specify **linear alchemical schedule**, **-log** complex directories specify **logistic alchemical schedule with softplus parameters**.)*

```
cd $HOME/water-droplet/complexes/EtOH-lin
./runopenmm $HOME/devel/async_re-openmm/bedamtempt_async_re.py water-droplet-ethanol_asyncre.cntl
```

Each RE simulation is set to run for 480 mins (8 hours). You can change the duration of the simulation run by changing the ```WALL_TIME``` in the ```*_asyncre.cntl``` file present in each complex directory.
Monitor the progress of each ASyncRE simulation by inspecting the ```*_stat.txt``` file. For example:

```
cd $HOME/water-droplet/complexes/EtOH-lin
cat water-droplet-ethanol_stat.txt
```
Each replica runs in a separate subdirectory named ```r0```, ```r1```, etc. Each cycle generates ```.out```, ```.dms```, ```.log```, ```.pdb```, and ```.dcd``` files. For example the binding energy samples for the first cycle of replica 2 are stored in the file ```r2/water-droplet-ethanol_1.out``` . The .dms files are used to start the following cycle. The .pdb and .dcd files are used for trajectory visualization.
 
The ASyncRE process can be killed at any time with ```^C``` and optionally restarted. However, replicas currently running on remote machines are likely to keep running and may have to be killed before ASyncRE can be restarted. To start from scratch (that is from the first cycle) remove the replicas directories by doing ```rm -r r? r??```.








