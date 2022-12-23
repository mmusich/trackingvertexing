---
title: Setup
---
# Set-up your working environment

> ## Important
> **This exercise is meant to be run from cmslpc-sl7.fnal.gov or lxplus.cern.ch, but we will prefer the first one.**
{: .callout}
### Logging in
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
### Accessing the data

We will be using ZeroBias events (events from nominally colliding bunch crossings but without a requirement for any specific activity in the event) from Run 2 (2018) data. This dataset is small enough to be easily accessible as a file. You should have plenty of space, copy it to your working directory with the copy command below:

~~~
xrdcp root://cmseos.fnal.gov//store/user/cmsdas/2023/short_exercises/trackingvertexing/run321167_ZeroBias_AOD.root .
~~~
{: .language-bash}
### Checking the file content

You can check the content of the file by running the simple script as follows
~~~
edmDumpEventContent run321167_ZeroBias_AOD.root
~~~
{: .language-bash}

> ## Bonus Python hints!
> Note that the brilconda3 virtual environment offers a faily recent version of Python (3.7) and batteries are included (lots of third-party Python packages)!
> You might consider defining a `brilconda3` alias in your `~/.bashrc` to prepend brilconda3 to the `$PATH` environment variable:
> ```bash
> echo 'alias brilconda3="[[ -d /cvmfs/cms-bril.cern.ch/brilconda3 ]] && export PATH=/cvmfs/cms-bril.cern.ch/brilconda3/bin:${PATH}"' >> "${HOME}/.bashrc"
> ```
> {: .source}
> You might also find it useful to set up a brilconda3-based Python3 virtual environment (each virtual environment lives "portably" inside a directory; it includes its own Python binary and can have an independent set of Python packages).
The `--system-site-packages` flag gives your virtual environment access to the brilconda3 packages.
> ```bash
> /cvmfs/cms-bril.cern.ch/brilconda3/bin/python3 -m venv --system-site-packages "${HOME}/.local/brilconda3"
> source "${HOME}/.local/brilconda3/bin/activate"
> command -v python3
> python3 -m pip freeze | awk 'BEGIN{FS="="}; {printf("%s ", $1)} END{print}'
> ```
> {: .source}
> ```
> "${HOME}/.local/brilconda3/bin/python3"
> beautifulsoup4 brilws certifi cffi chardet cheroot CherryPy click conda conda-build conda-package-handling contextlib2 cryptography cx-Oracle cycler Cython docopt filelock Flask Flask-SocketIO glob2 h5py idna itsdangerous jaraco.functools Jinja2 kiwisolver libarchive-c MarkupSafe matplotlib mock more-itertools numexpr numpy pandas pkginfo portend prettytable psutil pycosat pycparser pyOpenSSL pyparsing PySocks python-dateutil python-engineio python-socketio pytz PyYAML pyzmq requests ruamel-yaml schema scipy sip six soupsieve SQLAlchemy tables tempora tornado tqdm urllib3 Werkzeug
> ```
> {: .output}
{: .solution}
> ### Solution
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
