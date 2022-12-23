---
title: Setup
---
# Set-up your working environment

> ## Important
> **This exercise is meant to be run from cmslpc-sl7.fnal.gov.**
{: .callout}
## Logging in
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

# crab, we will not use it but take it as an habit!                                                                                                    
source /cvmfs/cms.cern.ch/crab3/crab.sh

# create your CMSSW working area
mkdir TrackingShortExercize
cd TrackingShortExercize
cmsrel CMSSW_10_6_18
cd CMSSW_10_6_18/src
cmsenv

# we will work in the source directory all the time!
cd -
~~~
{: .language-bash}
If you have a GitHub account, you can also do the following:

~~~
# set up your full name
git config --global user.name '<your name> <your last name>'

# set up your email
git config --global user.email '<your e-mail>'

# set up your GitHub user name
git config --global user.github <your github username>
~~~
{: .language-bash}

## Accessing the data

We will be using ZeroBias events (events from nominally colliding bunch crossings but without a requirement for any specific activity in the event) from Run 2 (2018) data. This dataset is small enough to be easily accessible as a file. You should have plenty of space, copy it to your working directory with the copy command below:

~~~
xrdcp root://cmseos.fnal.gov//store/user/cmsdas/2023/short_exercises/trackingvertexing/run321167_ZeroBias_AOD.root .
~~~
{: .language-bash}

## Checking the file content

You can check the content of the file by running the simple script as follows

~~~
edmDumpEventContent run321167_ZeroBias_AOD.root
~~~
{: .language-bash}

{% include links.md %}
