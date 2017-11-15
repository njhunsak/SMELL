# VistaSoft

## MATLAB Function

```
vi ~/scripts/SMELL/subjID.m
```

Copy and paste:

```
%%%%%%%
function subjID(x)

    % Display participant ID:
    display(x);

    % Get home directory:
    var = getenv('HOME');

    % Add modules to MATLAB. Do not change the order of these programs:
    SPM8Path = [var, '/apps/matlab/spm8'];
    addpath(genpath(SPM8Path));
    vistaPath = [var, '/apps/matlab/vistasoft'];
    addpath(genpath(vistaPath));
    AFQPath = [var, '/apps/matlab/AFQ'];
    addpath(genpath(AFQPath));

    % Set file names:
    subjDir = [var, '/compute/images/SMELL/', x];
    dtiFile = [subjDir, '/dti/dwi.nii.gz'];
    cd (subjDir);

    % Don't change the following code:
    ni = readFileNifti(dtiFile);
    ni = niftiSetQto(ni, ni.sto_xyz);
    writeFileNifti(ni, dtiFile);

    dwParams = dtiInitParams('rotateBvecsWithCanXform',1,'phaseEncodeDir',2,'clobber',1);

    % Here's the one line of code to do the DTI preprocessing:
    dtiInit(dtiFile, 'MNI', dwParams);

    % Clean up files and exit:
    movefile('dwi_*', 'dti/');
    movefile('dtiInitLog.mat', 'dti/');
    movefile('ROIs', '*trilin/');

    exit;
```

## Job Script

Create job script:

``` bash
vi ~/scripts/SMELL/dtiInit_job.sh
```

Copy and paste:

``` bash
#!/bin/bash

#SBATCH --time=04:00:00   # walltime
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
matlab -nodisplay -nojvm -nosplash -r "subjID('$1')"
```

## Batch Script

``` bash
vi ~/scripts/SMELL/dtiInit_batch.sh
```

Copy and paste:

``` bash
#!/bin/bash
for subj in $(ls ~/compute/images/SMELL/); do
sbatch \
-o ~/logfiles/${1}/output_${subj}.txt \
-e ~/logfiles/${1}/error_${subj}.txt \
~/scripts/SMELL/dtiInit_job.sh \
${subj}
sleep 1
done
```

## Submit Batch Script

``` bash
var=`date +"%Y%m%d-%H%M%S"`
mkdir -p ~/logfiles/${var}
sh ~/scripts/SMELL/dtiInit_batch.sh $var
```

## Check Motion

Change the code of the original file `dtiCheckMotion.m`

``` matlab
function [fh, figurename] = dtiCheckMotion(ecXformFile,visibility)
% Plots rotations and translations from an eddy-current correction
% transform file.
%
%   [fh, figurename]  = dtiCheckMotion([ecXformFile=uigetfile],visibility)
%
% INPUTS:
%   ecXformFile - Eddy Current correction trasformation infromation. This
%                 file is generally generated and saved by dtiInit.m. See
%                 dtiInit.m and also dtiRawRohdeEstimateEddyMotion.m
%
%   visibility  - A figure with the estimates of rotation and translation
%                 will be either displayed (visibility='on') or not
%                 (visibility='off'). The figure will be always saved in
%                 the same directory of the ecXformFile.mat file.
%
% OUTPUTS:
%   fh - Handle to the figure of the motion estimae. Note, this figure
%        might be set to invisible using 'visibility'. To display the
%        figure invoke the following command: set(fh,'visibility','on').
%
%   figurename - Full path to the figure saved out to disk showing the
%                motion estimates.
%i
% Franco Pestilli & Bob Dougherty Stanford University

if notDefined('visibility'), visibility = 'off'; end
if(~exist('ecXformFile','var') || isempty(ecXformFile))
   [f,p] = uigetfile('*.mat','Select the ecXform file');
   if(isequal(f,0)), disp('Canceled.'); retun; end
   ecXformFile = fullfile(p,f);
end

% Load the stored trasformation file.
ec = load(ecXformFile);
t = vertcat(ec.xform(:).ecParams);

% Generate dataset that contains the motion correct for each volume - NJH
% 2016-04-05
mc = t(:,1:6);
mc(:,4:6) = (mc(:,4:6)/(2*pi)*360);
mc=dataset({mc 'x' 'y' 'z' 'pitch' 'roll' 'yaw'});

% We make a plot of the motion correction during eddy current correction
% but we do not show the figure. We only save it to disk.
fh = mrvNewGraphWin([],[],visibility);
if isstruct(fh)
    fh = fh.Number;
end
subplot(2,1,1);
plot(t(:,1:3));
title('Translation');
xlabel('Diffusion image number (diffusion direction)');
ylabel('translation (voxels)');
legend({'x','y','z'});

subplot(2,1,2);
plot(t(:,4:6)/(2*pi)*360);
title('Rotation');
xlabel('Diffusion image number (diffusion direction)');
ylabel('rotation (degrees)');
legend({'pitch','roll','yaw'});

% Save out a PNG figure with the same filename as the Eddy Currents correction xform.
[p,f,~] = fileparts(ecXformFile);
figurename = fullfile(p,[f,'.png']);

% Added export csv file of motion correction - NJH 2016-04-05
filename = fullfile(p,[f,'.csv']);
export(mc,'File',filename,'Delimiter',',');

% Old Code
% printCommand = sprintf('print(%s, ''-painters'',''-dpng'', ''-noui'', ''%s'')', ...
% num2str(fh),figurename);
% New Code - NJH 2016-04-05
printCommand = sprintf('saveas(fh,figurename)');

fprintf('[%s] Saving Eddy Current correction figure: \n %s \n', ...
         mfilename,figurename);
eval(printCommand);

% Delete output if it was not requested
if (nargout < 1), close(fh); clear fh figurename; end

return;
```

Open MATLAB:

``` bash
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

i = dir([var,'/compute/images/SMELL/']);
for x = 3:size(i,1),
	ecFile = [var,'/compute/images/SMELL/',i(x).name,'/dti/dwi_aligned_trilin_ecXform.mat'];
	dtiCheckMotion(ecFile);
end
exit;
```

``` bash
mkdir -p ~/compute/analyses/SMELL/motion/
for i in $(find ~/compute/images/SMELL/ -name "dwi_aligned_trilin_ecXform.csv"); do
	subjid=$(basename $(dirname $(dirname $i)));
	cp -v $i ~/compute/analyses/SMELL/motion/${subjid}.csv;
done
module load r
R
```
When R opens:

``` r
projDir="~/compute/analyses/SMELL/motion"
files=list.files(projDir,full.names=TRUE)
mydata=data.frame()
for (i in 1:length(files))
{
	temp=read.csv(files[i],sep=",",header=TRUE)
	avgtranslation=mean(as.matrix(temp[c(1:3)]))
	avgrotation=mean(as.matrix(temp[c(4:6)]))
	subjID=files[i]
	tempid=strsplit(as.character(subjID),"/")
	subjID=sapply(tempid,"[[",8)
	tempid=strsplit(as.character(subjID),"[.]")
	subjID=sapply(tempid,"[[",1)
	temp=cbind(subjID,avgtranslation,avgrotation)
	mydata=rbind(mydata,temp)
}
write.table(mydata,"~/compute/analyses/SMELL/motion/motion.csv",row.names=F,sep=",")
q()
```
