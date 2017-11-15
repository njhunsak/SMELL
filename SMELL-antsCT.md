# ANTs Cortical Thickness

## Job Script

Create job script:

``` bash
vi ~/scripts/SMELL/antsCT_OASIS30_job.sh
```

Copy and paste:

``` bash
#!/bin/bash

#SBATCH --time=06:00:00   # walltime
#SBATCH --ntasks=1   # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=16384M  # memory per CPU core

# Compatibility variables for PBS. Delete if not needed.
export PBS_NODEFILE=`/fslapps/fslutils/generate_pbs_nodefile`
export PBS_JOBID=$SLURM_JOB_ID
export PBS_O_WORKDIR="$SLURM_SUBMIT_DIR"
export PBS_QUEUE=batch


# Set the max number of threads to use for programs using OpenMP.
export OMP_NUM_THREADS=$SLURM_CPUS_ON_NODE

# LOAD ENVIRONMENTAL VARIABLES
var=`id -un`
export ANTSPATH=/fslhome/$var/apps/ants/bin/
PATH=${ANTSPATH}:${PATH}

# INSERT CODE, AND RUN YOUR PROGRAMS HERE
DATA_DIR=~/compute/images/SMELL/${1}/
TEMPLATE_DIR=~/templates/OASIS-30_Atropos_template/
mkdir -p ~/compute/analyses/SMELL/antsCT_OASIS30/${1}/
~/apps/ants/bin/antsCorticalThickness.sh \
-d 3 \
-a ${DATA_DIR}/str/t1.nii.gz \
-e ${TEMPLATE_DIR}/T_template0.nii.gz \
-t ${TEMPLATE_DIR}/T_template0_BrainCerebellum.nii.gz \
-m ${TEMPLATE_DIR}/T_template0_BrainCerebellumProbabilityMask.nii.gz \
-f ${TEMPLATE_DIR}/T_template0_BrainCerebellumExtractionMask.nii.gz \
-p ${TEMPLATE_DIR}/Priors2/priors%d.nii.gz \
-q 1 \
-o ~/compute/analyses/SMELL/antsCT_OASIS30/${1}/
```

## Batch Script

Create batch script:

``` bash
vi ~/scripts/SMELL/antsCT_OASIS30_batch.sh
```

Copy and paste:

``` bash
#!/bin/bash

for subj in $(ls ~/compute/images/SMELL/); do
sbatch \
-o ~/logfiles/${1}/output_${subj}.txt \
-e ~/logfiles/${1}/error_${subj}.txt \
~/scripts/SMELL/antsCT_OASIS30_job.sh \
${subj}
sleep 1
done
```

## Submit Jobs

``` bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/$var
sh ~/scripts/SMELL/antsCT_OASIS30_batch.sh $var
```

## Sync Data

``` bash
rsync -rauv \
intj5@ssh.fsl.byu.edu:/fslhome/intj5/compute/analyses/SMELL/antsCT_OASIS30 \
/Volumes/data/analyses/SMELL/
```