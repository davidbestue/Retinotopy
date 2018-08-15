# Retinotopy

<br/>
<br/>

## Process the structural images

+ Start from the raw .dcm files

In our example, the .dcm files are in a folder called ** str_raw **

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
** You need to run this everytime you use freesurfer **

```
export SUBJECTS_DIR=/home/david/Desktop/freesurfer/David/structurals/struct_1
cd $SUBJECTS_DIR

```

<br/>
<br/>



** Before starting with the retinotopy you have to be carefull about the structure of the directories **

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

<br/>

+ What is a rtopy.par?

It is an empty document that has the information about the kind of task (an example is attached) 
[how to create them? ] (https://surfer.nmr.mgh.harvard.edu/fswiki/FsFastIndividualRetinotopyAnalysis)

```
Options of the rtopy.par:

stimtype eccen or polar
direction pos or neg

```
<br/>

# Run the preprocess of the data
<br/>

From the retinotopy directory run the preprocessing

[More details about preprocessing in freesurfer](https://surfer.nmr.mgh.harvard.edu/fswiki/FsFastTutorialV5.1/FsFastPreProc)

```
preproc-sess -s David -fwhm 5 -fsd bold -per-run

```
<br/>







 
 
 
 
