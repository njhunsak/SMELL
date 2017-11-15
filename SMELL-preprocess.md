# Preprocess

## DICOM to NiFTI

Convert T1 image:

``` bash
for i in $(find /Volumes/mado/ -type d -name "t1_mpr_sag_iso*"); do
subjid=$(basename $(dirname $(dirname $i)))
mkdir -p ~/Desktop/SMELL/${subjid:9:13}/str/
cd ~/Desktop/SMELL/${subjid:9:13}/
~/Applications/dcm2niix/bin/dcm2niix -o str/ -x y -f t1 -z n $i
done
```

Covert DTI image:

``` bash
for i in $(find /Volumes/mado -type d -name "ep2d*(DTI_No_Angle)_[0-9][0-9]"); do
subjid=$(basename $(dirname $(dirname $i)))
mkdir -p ~/Desktop/SMELL/${subjid:9:13}/dti/
cd ~/Desktop/SMELL/${subjid:9:13}/
~/Applications/dcm2niix/bin/dcm2niix -o dti/ -x y -f dwi -z y $i
done
```

``` bash
for i in $(find /Volumes/mado -type d -name "ep2d*(DTI_No_Angle)_[0-9]"); do
subjid=$(basename $(dirname $(dirname $i)))
mkdir -p ~/Desktop/SMELL/${subjid:9:13}/dti/
cd ~/Desktop/SMELL/${subjid:9:13}/
~/Applications/dcm2niix/bin/dcm2niix -o dti/ -x y -f dwi -z y $i
done
```

## Sync Data

``` bash
rsync -rauv \
--exclude=".*" \
/Volumes/data/images/SMELL/ \
intj5@ssh.fsl.byu.edu:~/compute/images/SMELL/
```

## Job Script

Create script:

``` bash
vi ~/scripts/SMELL/preprocess_job.sh
```

Copy and paste:

``` bash
#!/bin/bash

#SBATCH --time=00:15:00   # walltime
#SBATCH --ntasks=1   # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=32768M  # memory per CPU core

# COMPATABILITY VARIABLES FOR PBS. DO NO DELETE.
export PBS_NODEFILE=`/fslapps/fslutils/generate_pbs_nodefile`
export PBS_JOBID=$SLURM_JOB_ID
export PBS_O_WORKDIR="$SLURM_SUBMIT_DIR"
export PBS_QUEUE=batch
export OMP_NUM_THREADS=$SLURM_CPUS_ON_NODE

# LOAD ENVIRONMENTAL VARIABLES
var=`id -un`
export ANTSPATH=/fslhome/${var}/apps/ants/bin/
PATH=${ANTSPATH}:${PATH}

# INSERT CODE, AND RUN YOUR PROGRAMS HERE
DATA_DIR=~/compute/images/SMELL/${1}/

## ACPC ALIGN
~/apps/art/acpcdetect \
-M \
-o ${DATA_DIR}/str/acpc.nii \
-i ${DATA_DIR}/str/t1.nii

## CROP
~/apps/c3d/bin/c3d \
${DATA_DIR}/str/acpc.nii \
-trim 20vox \
-o ${DATA_DIR}/str/cropped.nii.gz

## N4 BIAS FIELD CORRECTION
~/apps/ants/bin/N4BiasFieldCorrection \
-v -d 3 \
-i  ${DATA_DIR}/str/cropped.nii.gz \
-o ${DATA_DIR}/str/n4.nii.gz \
-s 4 -b [200] -c [50x50x50x50,0.000001]

## RESAMPLE
~/apps/c3d/bin/c3d \
${DATA_DIR}/str/n4.nii.gz \
-resample-mm 1x1x1mm \
-o ${DATA_DIR}/str/resampled.nii.gz
```

## Batch Script

Create batch script:

``` bash
vi ~/scripts/SMELL/preprocess_batch.sh
```

Copy and paste code:

``` bash
#!/bin/bash

for subj in $(ls ~/compute/images/SMELL/); do
  sbatch \
  -o ~/logfiles/${1}/output_${subj}.txt \
  -e ~/logfiles/${1}/error_${subj}.txt \
  ~/scripts/SMELL/preprocess_job.sh \
  ${subj}
  sleep 1
done
```

## Submit Jobs

``` bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/$var
sh ~/scripts/SMELL/preprocess_batch.sh $var
```

## Sync Data

``` bash
rsync \
-rauv \
intj5@ssh.fsl.byu.edu:~/compute/images/SMELL/ \
/Volumes/data/images/SMELL/
```

## Clean Up Directory

``` bash
find ~/compute/images/SMELL/ -type f -name "*.ppm" -exec rm {} \;
find ~/compute/images/SMELL/ -type f -name "t1.nii" -exec rm {} \;
find ~/compute/images/SMELL/ -type f -name "t1_Crop_1.nii" -exec rm {} \;
find ~/compute/images/SMELL/ -type f -name "n4.nii.gz" -exec rm {} \;
find ~/compute/images/SMELL/ -type f -name "acpc.nii" -exec rm {} \;
find ~/compute/images/SMELL/ -type f -name "t1_ACPC.txt" -exec rm {} \;
find ~/compute/images/SMELL/ -type f -name "cropped.nii.gz" -exec rm {} \;
for i in $(find ~/compute/images/SMELL/ -type f -name "resampled.nii.gz"); do
  cd $(dirname $i)
  mv $i $(dirname $i)/t1.nii.gz
done
```

## Final Sync

``` bash
rsync \
-rauv \
intj5@ssh.fsl.byu.edu:~/compute/images/SMELL/ \
/Volumes/data/images/SMELL/ \
--delete
```
