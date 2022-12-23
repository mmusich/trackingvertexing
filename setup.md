---
title: Setup
---
# Set-up your working environment

> ## Important
> **This exercise is meant to be run from cmslpc-sl7.fnal.gov.**
{: .callout}


~~~
# login to the LPC cluster with DISPLAY set
ssh -Y <USERNAME>@cmslpc-sl7.fnal.gov

# get to your no-backup working area
cd nobackup
 
# line to find out what shell you are in
echo $0
 
# CMSSW compiler                              
# if you are using BASH :                                                                      
export SCRAM_ARCH=slc7_amd64_gcc700

# if you are using (T)CSH
setenv SCRAM_ARCH slc7_amd64_gcc700

# crab                                                                                                    
source /cvmfs/cms.cern.ch/crab3/crab.sh

# create your CMSSW working area
mkdir TrackingShortExercize
cd TrackingShortExercize
cmsrel CMSSW_10_6_18
cd CMSSW_10_6_18/src
cmsenv
cd -

~~~
{: .language-bash}

{% include links.md %}
