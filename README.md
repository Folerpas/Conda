# How to create conda environment in HPC

## Step1: Create the environment

Check the modules available in the HPC, we are going to need Anaconda

```
module avail anaconda
```

Upload the available anaconda version, for me it was Anaconda3

```
module load Anaconda3
```

I created an environment to use Python. With Python I want to read netCDF data and generate figures with the library matplotlib

```
conda create -y -n py36 python=3.6 netcdf4 matplotlib=3.2
```

## Step2: Activate the environment

Activate my new environment

```
conda activate py36
```

Check if the correct Python version is installed in my environment

```
python --version
```

## Step3: Install libraries inside the environment

I am installing some libraries needed to proper execute my scripts :)

```
conda install -c anaconda ipython
conda update -n base -c defaults conda
conda install basemap
```
