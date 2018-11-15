# Retinotopy

<br/>

## Process the structural images

+ Start from the raw .dcm files

In our example, the .dcm files are in a folder called **str_raw**

You need the following structure

```
SUBJ01
    |--structural
            |--struct_1
                |--str_raw
                    00000001.dcm
                    ....
                    00000180.dcm
                
```

<br/>


Run the following code (I always run that in akalla)

```
tcsh
source /usr/local/freesurfer/SetUpFreeSurfer.csh
setenv SUBJECTS_DIR ~/Desktop/Distractor_project/imaging/David/structural/struct_1
cd $SUBJECTS_DIR

recon-all -i str_raw/00000001.dcm -subject David_struct1 -all

```

<br/>

As I want the structural data in my local I moved it from the server with scp with the following lines

```
scp -r davsan@akalla.cns.ki.se:~/Desktop/Distractor_project/imaging/David/structural/struct_1/David_struct1/. /home/david/Desktop/freesurfer/David/structurals/struct_1/d001
scp -r davsan@akalla.cns.ki.se:~/Desktop/Distractor_project/imaging/David/structural/struct_1/fsaverage /home/david/Desktop/freesurfer/David/structurals/struct_1
scp -r davsan@akalla.cns.ki.se:~/Desktop/Distractor_project/imaging/David/structural/struct_1/lh.EC_average /home/david/Desktop/freesurfer/David/structurals/struct_1
scp -r davsan@akalla.cns.ki.se:~/Desktop/Distractor_project/imaging/David/structural/struct_1/rh.EC_average /home/david/Desktop/freesurfer/David/structurals/struct_1
```


<br/>

Now you need to set the enviroment to link all the analysis you do in freesurfer to your structural
**You need to run this everytime you use freesurfer**

```
export SUBJECTS_DIR=/home/david/Desktop/freesurfer/David/structurals/struct_1
cd $SUBJECTS_DIR
cd /home/david/Desktop/freesurfer/retinotopy

export SUBJECTS_DIR= /mnt/c/Users/David/Desktop/KI_Desktop/freesurfer/r001/structurals/struct_1
cd $SUBJECTS_DIR
cd /mnt/c/Users/David/Desktop/KI_Desktop/freesurfer/r001/retinotopy

```

<br/>
<br/>



**Before starting with the retinotopy you have to be carefull about the structure of the directories**

[Info about directories and freesurfer](https://surfer.nmr.mgh.harvard.edu/fswiki/FsFastTutorialV5.1/FsFastDirStruct)

In this structure it is not important if the data was adquired in different sessions. All the run that you want
to include go inside the **bold** folder

<br/>

```
retinotopy
    |--sessid (empty file with the names of subjects (SUBJ01)
    |--SUBJ01
        |--subjectname (empty file with the name of the structural correspondence)
        |--bold
            |--001 (run)
                |--f.nii (functional file)
                |--rtopy.par
            |--002
                |--f.nii
                |--rtopy.par
               
 ```
 
The name of run folders has to be 3-digit with padding 0s. Something like “01″ or “1″ won’t work. Under run folders, put the functional imaging data “f.nii” and paradigm file (rtopy.par) 

The functional data has to be in .nii format. If the data that you get is in .dcm you can convert it to a 4D .nii format by using the matlab tool [dcm2nii](https://www.mathworks.com/matlabcentral/fileexchange/42997-xiangruili-dicm2nii)
After transformig the functional .dcm to .nii I moved it to the local

```
Example:

scp -r davsan@akalla.cns.ki.se:~/Desktop/Distractor_project/imaging/David/Att_map_tilted/run1/fmri_1_RetinoOpt_IPS_Tilt.nii /home/david/Desktop/freesurfer/retinotopy/David/bold/001/f.nii 

```

<br/>

+ **What is a rtopy.par?**

It is an empty document that has the information about the kind of task (an example is attached) 
[how to create them? ](https://surfer.nmr.mgh.harvard.edu/fswiki/FsFastIndividualRetinotopyAnalysis)

```
Options of the rtopy.par:

stimtype eccen or polar
direction pos or neg

```
<br/>

# Run the preprocessing of the data
<br/>

From the retinotopy directory run the preprocessing

[More details about preprocessing in freesurfer](https://surfer.nmr.mgh.harvard.edu/fswiki/FsFastTutorialV5.1/FsFastPreProc)

```
preproc-sess -s David -fwhm 5 -fsd bold -per-run
```
<br/>

***For the encoding and WMtask, we preprocessed the data with SPM. Check this other [repository](https://github.com/davidbestue/Preprocessing-functional-images)***

Some of the data was also preprocessed with freesurfer as it was needed to move the masks of the ROIs to a standard functional space.


# Make the analysis

[More details about 1st level analysis in freesurfer](https://surfer.nmr.mgh.harvard.edu/fswiki/FsFastTutorialV5.1/FsFastFirstLevel)

<br/>
Here you will specify the retinotopy parameters for the analysis
**You will run one analysis for each hemisphere**

- analysis : you give a name (it will generate a folder with this name, **add the hemisphere on the name**) 
- surface self (in the subject structure)
-TR : the time between two volumes 
-retinotopy : time of one cycle of the rotating edge or the eccentricity growth (it has to be the same)
-paradigm : name of the rtopy.par fiels
-fsd : folder where the functional data is
-fwhm 5 : smooth by 5mm FWHM after resampling (volume and surface separately)
-per-run: you run it in every run


```
mkanalysis-sess -analysis retinotopy.lh  -surface self lh -TR 1.6 -retinotopy 28.5 -paradigm rtopy.par -fsd bold -fwhm 5 -per-run
mkanalysis-sess -analysis retinotopy.rh  -surface self rh -TR 1.6 -retinotopy 28.5 -paradigm rtopy.par -fsd bold -fwhm 5 -per-run
```
<br/>


# Run the analysis

```
selxavg3-sess -analysis retinotopy.lh -s David
selxavg3-sess -analysis retinotopy.rh -s David
```

<br/>
<br/>


# Visualize the results

+ **Significance maps:**

```
tksurfer-sess -a retinotopy.rh -s David -tksurfer
tksurfer-sess -a retinotopy.lh -s David -tksurfer
```

![](https://github.com/davidbestue/Retinotopy/blob/master/imgs/sign_map.png)

<br/>


+ **Display raw angle:**

```
tksurfer-sess -a retinotopy.rh -s David -map angle -tksurfer
tksurfer-sess -a retinotopy.lh -s David -map angle -tksurfer
```

![](https://github.com/davidbestue/Retinotopy/blob/master/imgs/raw_angle.png)

<br/>

+ **Display angle masked by sig:**

```
tksurfer-sess -a retinotopy.rh -s David -map angle.masked -tksurfer
tksurfer-sess -a retinotopy.lh -s David -map angle.masked -tksurfer
```

![](https://github.com/davidbestue/Retinotopy/blob/master/imgs/angle_masked.png)

<br/>

<br/>


# Field sign map

[Follow this link to create an occipital patch](http://www.alivelearn.net/?p=65) 
What follows is the adapted code in my case and some "corrections"

An [alternative](https://surfer.nmr.mgh.harvard.edu/fswiki/FreeSurferOccipitalFlattenedPatch) for the same process. 

Cut occiput surface
Display the inflated left hemisphere.

```
tksurfer David lh inflated
```
Load curvature:
"File --> Curvature --> Load curvature"
"/home/david/Desktop/freesurfer/David/structurals/struct_1/David/surf/lh.curv"

Rotate the brain until the medial surface is facing you.
Then select points along calcarine fissure and press button “Cut line”.

![](https://github.com/davidbestue/Retinotopy/blob/master/imgs/cut_calcarine_fissure.png)

Select 3 points to define the cutting plane: 2 on medial side and 1 on lateral side. 
![](https://github.com/davidbestue/Retinotopy/blob/master/imgs/1_2.png)

Choose a 4th point to specify which portion of surface to keep and press button “Cut plane”.
![](https://github.com/davidbestue/Retinotopy/blob/master/imgs/3_4.png)

<br/>

Once it is cut

<br/>

![](https://github.com/davidbestue/Retinotopy/blob/master/imgs/cut.png)

<br/>
Save (File > Patch > Save As) as file lh.occip.patch.3d.
(To know how to cut full brain, check out the **fs_hands-on.pdf**, slides 36 and 37 )

<br/>
**Now you need to flatten this occipital patch**

cd to the subject’s “surf” directory and run

```
mris_flatten -w 0 -distances 20 7 lh.occip.patch.3d  lh.occip.patch.flat
```
Flattening takes 1-2 hours.

To visualize the patch, you can first load the subject’s inflated surface ('tksurfer David lh inflated')
then File > Patch > Load Patch …
(You also need to load the curvature on top)

![](https://github.com/davidbestue/Retinotopy/blob/master/imgs/occipital_patch_flat.png)


**Repeat the steps for the right hemisphere.**

Now you can run the field sign

```
fieldsign-sess -a retinotopy.lh -patch lh.occip.patch.flat -s David
fieldsign-sess -a retinotopy.rh -patch rh.occip.patch.flat -s David

If you want the whole brian you can run this:

fieldsign-sess -a retinotopy.rh -sphere -s David
fieldsign-sess -a retinotopy.lh -sphere -s David
```

**Visualize the results**

+ **Display field sign:**

```
tksurfer-sess -a retinotopy.lh -s David -fieldsign -tksurfer
```

+ **Patch**
<br/>

![](https://github.com/davidbestue/Retinotopy/blob/master/imgs/fsign.png)


+ **Whole (sphere)**
<br/>

![](https://github.com/davidbestue/Retinotopy/blob/master/imgs/sphere.png)

<br/>

**Now you need to create a mask over the regions of interest**


# Make a mask
<br/>

Follow the pdf attached **instructions_creating_mask.pdf**

**Depending from what image you want the mask**

```
Options:

cd /home/david/Desktop/freesurfer/retinotopy/David/bold/retinotopy.rh

The rtopy.par with polar to get the anges:
rtview --s David --rh --polar --real polar/real.nii.gz --imag polar/imag.nii.gz --fsig polar/fsig.nii.gz

The rtopy.par with eccen to get the radial:
rtview --s David --rh --eccen --real eccen/real.nii.gz --imag eccen/imag.nii.gz --fsig eccen/fsig.nii.gz


cd /home/david/Desktop/freesurfer/retinotopy

All together for significance
tksurfer-sess -a retinotopy.rh -s David -tksurfer

All together for angle
tksurfer-sess -a retinotopy.rh -s David -map angle -tksurfer

All together for angle masked by significance
tksurfer-sess -a retinotopy.rh -s David -map angle.masked -tksurfer

All together to make the significance map
tksurfer-sess -a retinotopy.lh -s David -fieldsign -tksurfer


There is a equivalence between:
rtview --s David --rh --polar --real polar/real.nii.gz --imag polar/imag.nii.gz --fsig polar/fsig.nii.gz
and
tksurfer-sess -a retinotopy.rh -s David -map angle -tksurfer
```

Once you have the image you will mark the region of interest

**1. Press the “Make Closed Path” button.**
<br/>
**2. Mark with the cursor points defining the area you want to delimitate.**
<br/>
**3. Press the “Make Closed Path” button.** 
<br/>
**4. Click with the cursor on a point inside the area.**
<br/>
**5. Click on “Custom Fill”.**
<br/>
Then select on:
<br/>
“Fill conditions” - “Up to and including paths”;
<br/>
“Fill From” - “Last clicked vertex”
<br/>
"Action” - Create new label
<br/>
press “Fill”
<br/>

**6. press File > Label > Save Selected Label**
<br/>
    Iput a name, for example “v1rh.label” and click ok.


**Now you need to adapt the mask to the voxel size of the functional image**

We use the [mri_vol2roi](https://surfer.nmr.mgh.harvard.edu/fswiki/mri_vol2roi) function

mri_vol2roi --label ***path***  --srcvol ***path*** --srcreg ***path*** --finalmskvol ***path*** --roiavg ***X.delet.dat***

--label : path to label file
<br/>
--srcvol : Bfloat/bshort stem of the volume from which the ROI is to be extracted.
<br/>
--srcreg : Registration between src volume and subject's anatomical (ie, a register.dat). This is only needed when using a label.
<br/>
--finalmskvol : Save the final set of voxels selected for the ROI in this volume.
<br/>
--roiavg : Save output as a bfloat 'volume'. This flag is actually necessary even if you are not going to use this output.

<br/>

Example:

Mask: /home/david/Desktop/freesurfer/David/structurals/struct_1/David/label/v1rh.label

Volume functional image (reference): /home/david/Desktop/freesurfer/David/encoding/encoding_1/bold/001/template.nii.gz
You get this image when you preprocess a functional image in freesurfer.

```
mri_vol2roi --label /home/david/Desktop/freesurfer/David/structurals/struct_1/David/label/v1rh.label --srcvol /home/david/Desktop/freesurfer/David/encoding/encoding_1/bold/001/template.nii.gz --srcreg /home/david/Desktop/freesurfer/David/encoding/encoding_1/bold/001/register.dof6.dat~ --finalmskvol /home/david/Desktop/freesurfer/David/encoding/encoding_1/bold/001/maskv1rh --roiavg v1rh.delete.dat
```




 
 
 
 
