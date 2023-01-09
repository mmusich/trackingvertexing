---
title: Setup
---
# Set-up your working environment


> ## Mattermost (chat)
> There is a dedicated Mattermost team, called [CMSDAS@LPC 2023](https://mattermost.web.cern.ch/signup_user_complete/?id=xywbon1zutbqj8a53rzqomrp9r), setup to facilitate communication and discussions via live chat (which is also archived). The channel is hosted by the [CERN Mattermost instance](https://mattermost.web.cern.ch/).
> 
> If you have never used Mattermost at CERN, please know that you will need your CERN login credentials (SSO) and you will need to join the private CMSDAS@LPC 2023 team in order to be able to see (or find using the search channels functionality) the channels setup for communications related to the school.
> 
> If you already have used Mattermost at CERN, please know that when you click direct links to channels (as you will find below) that are within the CMSDAS@LPC 2023 team, you **may** be redirected to the last Mattermost team you used. If this happens, remember to click the [signup link to join the CMSDAS@LPC 2023 team](https://mattermost.web.cern.ch/signup_user_complete/?id=xywbon1zutbqj8a53rzqomrp9r) to switch to the correct team from which you should be able to see the individual channels. If that still doesn’t work, remove all cookies associated with cern.ch and restart your browser.
> 
> The [ShortExTrackingVertexing](https://mattermost.web.cern.ch/cmsdaslpc2023/channels/shortextrackingvertexing) will be available once you join or switch to the CMSDAS@LPC 2023 team!
> 
> Note that you can access Mattermost via browser or you can download the corresponding application running standalone on your laptop or smartphone. The laptop application does not work with CERN certificate login.
{: .prereq}

> ## Important
> **This exercise is designed to run only on cmslpc-sl7 as copies of the scripts are present there.**
> See the [Connecting to the cmslpc cluster link](http://uscms.org/uscms_at_work/physics/computing/getstarted/uaf.shtml) helper page.
{: .callout}
## Logging in
Keep in mind your `<USERNAME>` and use it in the following instructions:
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

# set cmsenv and cmsrel aliases 

source /cvmfs/cms.cern.ch/cmsset_default.sh

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

We will be using **ZeroBias events** (events from nominally colliding bunch crossings but without a requirement for any specific activity in the event) from Run 2 (2018) data. This dataset is small enough to be easily accessible as a file (remember to be in the **nobackup** working area). You should have plenty of space, copy it to your working directory with the copy command below:

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
> ## Solution
> It will print a (very) long list of collections, in particular we are interested in the following lines (end of the list):
> ```
> Type                                  Module                      Label             Process   
> ----------------------------------------------------------------------------------------------
> vector<reco::Track>                   "generalTracks"             ""                "RECO"
> vector<float>                         "generalTracks"             "MVAValues"       "RECO"
> vector<reco::Vertex>                  "offlinePrimaryVertices"    ""                "RECO"
> vector<reco::Vertex>                  "offlinePrimaryVerticesWithBS"   ""           "RECO"
> vector<reco::VertexCompositeCandidate> "generalV0Candidates"      "Kshort"          "RECO"
> vector<reco::VertexCompositeCandidate> "generalV0Candidates"      "Lambda"          "RECO"
> vector<reco::Track>                   "globalMuons"               ""                "RECO"
> vector<reco::TrackExtra>              "globalMuons"               ""                "RECO"
> vector<reco::GsfTrack>                "electronGsfTracks"         ""                "RECO"
> ```
> {: .output}
{: .solution}

{% include links.md %}
