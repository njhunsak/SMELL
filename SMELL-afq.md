# AFQ

Create output directories:

``` bash
mkdir -p ~/compute/analyses/SMELL/AFQ
mkdir -p ~/compute/analyses/SMELL/AFQ-CC
```

## Set Parameters

Create script:

``` bash
vi ~/scripts/SMELL/afq_parameters.m
```

Copy and paste:

``` matlab
var = getenv('HOME');
sub_dirs = {[var, '/compute/images/SMELL/1048/dti30trilin/'], ...
[var, '/compute/images/SMELL/1063/dti30trilin/'], ...
[var, '/compute/images/SMELL/1086/dti30trilin/'], ...
[var, '/compute/images/SMELL/1108/dti30trilin/'], ...
[var, '/compute/images/SMELL/1113/dti30trilin/'], ...
[var, '/compute/images/SMELL/1116/dti30trilin/'], ...
[var, '/compute/images/SMELL/1118/dti30trilin/'], ...
[var, '/compute/images/SMELL/1120/dti30trilin/'], ...
[var, '/compute/images/SMELL/1126/dti30trilin/'], ...
[var, '/compute/images/SMELL/1128/dti30trilin/'], ...
[var, '/compute/images/SMELL/1130/dti30trilin/'], ...
[var, '/compute/images/SMELL/1133/dti30trilin/'], ...
[var, '/compute/images/SMELL/1135/dti30trilin/'], ...
[var, '/compute/images/SMELL/1137/dti30trilin/'], ...
[var, '/compute/images/SMELL/1140/dti30trilin/'], ...
[var, '/compute/images/SMELL/608/dti30trilin/'], ...
[var, '/compute/images/SMELL/618/dti30trilin/'], ...
[var, '/compute/images/SMELL/632/dti30trilin/'], ...
[var, '/compute/images/SMELL/670/dti30trilin/'], ...
[var, '/compute/images/SMELL/676/dti30trilin/'], ...
[var, '/compute/images/SMELL/694/dti30trilin/'], ...
[var, '/compute/images/SMELL/696/dti30trilin/'], ...
[var, '/compute/images/SMELL/699/dti30trilin/'], ...
[var, '/compute/images/SMELL/701/dti30trilin/'], ...
[var, '/compute/images/SMELL/704/dti30trilin/'], ...
[var, '/compute/images/SMELL/706/dti30trilin/'], ...
[var, '/compute/images/SMELL/709/dti30trilin/'], ...
[var, '/compute/images/SMELL/711/dti30trilin/'], ...
[var, '/compute/images/SMELL/915/dti30trilin/'], ...
[var, '/compute/images/SMELL/976/dti30trilin/'], ...
[var, '/compute/images/SMELL/1055/dti30trilin/'], ...
[var, '/compute/images/SMELL/1065/dti30trilin/'], ...
[var, '/compute/images/SMELL/1104/dti30trilin/'], ...
[var, '/compute/images/SMELL/1109/dti30trilin/'], ...
[var, '/compute/images/SMELL/1115/dti30trilin/'], ...
[var, '/compute/images/SMELL/1117/dti30trilin/'], ...
[var, '/compute/images/SMELL/1119/dti30trilin/'], ...
[var, '/compute/images/SMELL/1121/dti30trilin/'], ...
[var, '/compute/images/SMELL/1127/dti30trilin/'], ...
[var, '/compute/images/SMELL/1129/dti30trilin/'], ...
[var, '/compute/images/SMELL/1131/dti30trilin/'], ...
[var, '/compute/images/SMELL/1134/dti30trilin/'], ...
[var, '/compute/images/SMELL/1136/dti30trilin/'], ...
[var, '/compute/images/SMELL/1138/dti30trilin/'], ...
[var, '/compute/images/SMELL/606/dti30trilin/'], ...
[var, '/compute/images/SMELL/613/dti30trilin/'], ...
[var, '/compute/images/SMELL/625/dti30trilin/'], ...
[var, '/compute/images/SMELL/656/dti30trilin/'], ...
[var, '/compute/images/SMELL/674/dti30trilin/'], ...
[var, '/compute/images/SMELL/683/dti30trilin/'], ...
[var, '/compute/images/SMELL/695/dti30trilin/'], ...
[var, '/compute/images/SMELL/698/dti30trilin/'], ...
[var, '/compute/images/SMELL/700/dti30trilin/'], ...
[var, '/compute/images/SMELL/703/dti30trilin/'], ...
[var, '/compute/images/SMELL/705/dti30trilin/'], ...
[var, '/compute/images/SMELL/708/dti30trilin/'], ...
[var, '/compute/images/SMELL/710/dti30trilin/'], ...
[var, '/compute/images/SMELL/712/dti30trilin/'], ...
[var, '/compute/images/SMELL/930/dti30trilin/'], ...
[var, '/compute/images/SMELL/999/dti30trilin/']};
sub_group = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...
	0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...
	0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...
	1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
	1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
	1, 1, 1, 1, 1, 1, 1, 1, 1, 1];
save('~/compute/analyses/SMELL/AFQ/sub_dirs.mat', 'sub_dirs');
save('~/compute/analyses/SMELL/AFQ/sub_group.mat', 'sub_group');
quit
```

Run script:

``` bash
cd ~/scripts/SMELL/
module load matlab
matlab -nodisplay -nojvm -nosplash -r afq_parameters
module unload matlab
```

## Job Script

Create job script:

``` bash
vi ~/scripts/SMELL/afq_job.sh
```

Copy and paste:

``` bash
#!/bin/bash

#SBATCH --time=50:00:00   # walltime
#SBATCH --ntasks=1  # number of processor cores (i.e. tasks)
#SBATCH --nodes=1   # number of nodes
#SBATCH --mem-per-cpu=24576M   # memory per CPU core

# Compatibility variables for PBS. Delete if not needed.
export PBS_NODEFILE=`/fslapps/fslutils/generate_pbs_nodefile`
export PBS_JOBID=$SLURM_JOB_ID
export PBS_O_WORKDIR="$SLURM_SUBMIT_DIR"
export PBS_QUEUE=batch

# Set the max number of threads to use for programs using OpenMP.
export OMP_NUM_THREADS=$SLURM_CPUS_ON_NODE

# LOAD MODULES, INSERT CODE, AND RUN YOUR PROGRAMS HERE
cd ~/scripts/SMELL/
module load matlab/r2013b
matlab -nodisplay -nojvm -nosplash -r afq_analysis
```

## MATLAB Script

Create script:

``` bash
vi ~/scripts/SMELL/afq_analysis.m
```

Copy and paste:

``` matlab
% Get home directory:
var = getenv('HOME');

% Add modules to MATLAB. Do not change the order of these programs:
SPM8Path = [var, '/apps/matlab/spm8'];
addpath(genpath(SPM8Path));
vistaPath = [var, '/apps/matlab/vistasoft'];
addpath(genpath(vistaPath));
AFQPath = [var, '/apps/matlab/AFQ'];
addpath(genpath(AFQPath));

load ~/compute/analyses/SMELL/AFQ/sub_dirs.mat
load ~/compute/analyses/SMELL/AFQ/sub_group.mat
outdir = fullfile([var, '/compute/analyses/SMELL/AFQ/']);
outname = fullfile(outdir, ['afq_' datestr(now, 'yyyy_mm_dd_HHMM')]);
afq = AFQ_Create(...
    'sub_dirs', sub_dirs, ...
    'sub_group', sub_group, ...
    'showfigs', false);
[afq, patient_data, control_data, norms, abn, abnTracts] = AFQ_run(...
    sub_dirs, ...
    sub_group, ...
    afq);
save(outname, 'afq');
```

## Submit Job Script

``` bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/${var}
sbatch \
-o ~/logfiles/${var}/ouput.txt \
-e ~/logfiles/${var}/error.txt \
~/scripts/SMELL/afq_job.sh
```

## Extract Data

``` bash
mkdir -p ~/compute/analyses/SMELL/AFQ/raw
module load matlab
matlab
```

When MATLAB opens:

``` bash
var = getenv('HOME');
SPM8Path = [var, '/apps/matlab/spm8'];
addpath(genpath(SPM8Path));
vistaPath = [var, '/apps/matlab/vistasoft'];
addpath(genpath(vistaPath));
AFQPath = [var, '/apps/matlab/AFQ'];
addpath(genpath(AFQPath));
```

Save data:

``` bash
load ~/compute/analyses/SMELL/AFQ/afq_2016_12_03_0722.mat
fgname={'left_thalamic_radiation', 'right_thalamic_radiation', ...
    'left_corticospinal', 'right_corticospinal', ...
    'left_cingulum_cingulate', 'right_cingulum_cingulate', ...
    'left_cingulum_hippocampus', 'right_cingulum_hippocampus', ...
    'callosum_forceps_major', 'callosum_forceps_minor', ...
    'left_ifof', 'right_ifof', 'left_ilf', 'right_ilf', ...
    'left_slf', 'right_slf', 'left_uncinate', 'right_uncinate', ...
    'left_arcuate', 'right_arcuate'};
outdir=fullfile('~/compute/analyses/SMELL/AFQ/raw/');
for n = 1:20
    csvwrite(fullfile(outdir, ...
        strcat('patient_',fgname{n},'_fa.csv')), ...
        afq.patient_data(n).FA)
    csvwrite(fullfile(outdir, ...
        strcat('patient_',fgname{n},'_rd.csv')), ...
        afq.patient_data(n).RD)
    csvwrite(fullfile(outdir, ...
        strcat('patient_',fgname{n},'_md.csv')), ...
        afq.patient_data(n).MD)
    csvwrite(fullfile(outdir, ...
        strcat('patient_',fgname{n},'_ad.csv')), ...
        afq.patient_data(n).AD)
    csvwrite(fullfile(outdir, ...
        strcat('control_',fgname{n},'_fa.csv')), ...
        afq.control_data(n).FA)
    csvwrite(fullfile(outdir, ...
        strcat('control_',fgname{n},'_rd.csv')), ...
        afq.control_data(n).RD)
    csvwrite(fullfile(outdir, ...
        strcat('control_',fgname{n},'_md.csv')), ...
        afq.control_data(n).MD)
    csvwrite(fullfile(outdir, ...
        strcat('control_',fgname{n},'_ad.csv')), ...
        afq.control_data(n).AD)
end
quit
```

## Make Demographic Files

``` bash
vi ~/compute/analyses/SMELL/AFQ/control.csv
```

Copy and paste:

```bash
1048
1063
1086
1108
1113
1116
1118
1120
1126
1128
1130
1133
1135
1137
1140
608
618
632
670
676
694
696
699
701
704
706
709
711
915
976
```

``` bash
vi ~/compute/analyses/SMELL/AFQ/patient.csv
```

Copy and paste:

```bash
1055
1065
1104
1109
1115
1117
1119
1121
1127
1129
1131
1134
1136
1138
606
613
625
656
674
683
695
698
700
703
705
708
710
712
930
999
```

## Add Subject ID

``` bash
cd ~/compute/analyses/SMELL/AFQ/
mkdir processed
for i in $(find ~/compute/analyses/SMELL/AFQ/raw/ -type f -name "patient*.csv"); do
	paste -d, <(cut -d, -f 1 patient.csv) $i \
	> ~/compute/analyses/SMELL/AFQ/processed/$(basename $i);
done
for i in $(find ~/compute/analyses/SMELL/AFQ/raw/ -type f -name "control*.csv"); do
	paste -d, <(cut -d, -f 1 control.csv) $i \
	> ~/compute/analyses/SMELL/AFQ/processed/$(basename $i);
done
```

## Preprocess CSV Files

``` bash
module load r
R
```

When R opens:

``` r
projDir="~/compute/analyses/SMELL/AFQ/"
rawdata=c("callosum_forceps_major","callosum_forceps_minor",
	"left_arcuate","right_arcuate",
	"left_cingulum_cingulate","right_cingulum_cingulate",
	"left_cingulum_hippocampus","right_cingulum_hippocampus",
	"left_corticospinal","right_corticospinal",
	"left_ifof","right_ifof",
	"left_slf","right_slf",
	"left_thalamic_radiation","right_thalamic_radiation",
	"left_uncinate","right_uncinate")
for (n in 1:length(rawdata))
{
	files=list.files(paste(projDir,"processed/",sep=""),
		pattern=rawdata[n],
		full.names=TRUE)
	mydata=data.frame()
	for (i in 1:length(files))
	{
  		temp=read.table(files[i],sep=",",header=FALSE)
  		temp$variable=files[i]
  		mydata=rbind(mydata,temp)
	}
	tempvar=strsplit(as.character(mydata$variable),"/")
	mydata$variable=sapply(tempvar,"[[",10)
	tempvar=strsplit(as.character(mydata$variable),"_")
	mydata$dtiscalar=sapply(tempvar,"[[",length(tempvar[[1]]))
	mydata$dtiscalar=substr(mydata$dtiscalar,1,2)
	tempid=strsplit(as.character(mydata$V1),"/")
	mydata$studyid=sapply(tempid,"[[",1)
	mydata$studyid=substr(mydata$studyid,1,4)
	mydata=mydata[c(104,103,2:101)]
	names(mydata)=c("studyid","dtiscalar","node1","node2","node3","node4","node5",
		"node6","node7","node8","node9","node10","node11","node12","node13",
		"node14","node15","node16","node17","node18","node19","node20","node21",
		"node22","node23","node24","node25","node26","node27","node28","node29",
		"node30","node31","node32","node33","node34","node35","node36","node37",
		"node38","node39","node40","node41","node42","node43","node44","node45",
		"node46","node47","node48","node49","node50","node51","node52","node53",
		"node54","node55","node56","node57","node58","node59","node60","node61",
		"node62","node63","node64","node65","node66","node67","node68","node69",
		"node70","node71","node72","node73","node74","node75","node76","node77",
		"node78","node79","node80","node81","node82","node83","node84","node85",
		"node86","node87","node88","node89","node90","node91","node92","node93",
		"node94","node95","node96","node97","node98","node99","node100")
	write.table(mydata,
		paste(projDir,"processed/",rawdata[n],".csv",sep=""),
		row.names=F,sep=",")
}
q()
```

## Clean Up Directories

``` bash
cd /fslhome/intj5/compute/analyses/SMELL/AFQ/processed
rm -rf control*
rm -rf patient*
cd /fslhome/intj5/compute/analyses/SMELL/AFQ/
rm -rf raw/
rm patient.csv
rm control.csv
mv processed/ data/
```

## Sync

``` bash
rsync -rauv --exclude="DICOMs" \
	intj5@ssh.fsl.byu.edu:~/compute/images/SMELL/ \
	/Volumes/data/images/SMELL/ \
	--delete
rsync -rauv \
	intj5@ssh.fsl.byu.edu:~/compute/analyses/SMELL/ \
	/Volumes/data/analyses/SMELL/ \
	--delete
```
