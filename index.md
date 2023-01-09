---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
venue: "LHC Physics Center, Fermi National Accelerator Laboratory"        # brief name of the institution that hosts the workshop without address (e.g., "Euphoric State University")
address: "In Person"      # full street address of workshop (e.g., "Room A, 123 Forth Street, Blimingen, Euphoria"), videoconferencing URL, or 'online'
country: "us"      # lowercase two-letter ISO country code such as "fr" (see https://en.wikipedia.org/wiki/ISO_3166-1#Current_codes) for the institution that hosts the workshop
language: "en"     # lowercase two-letter ISO language code such as "fr" (see https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) for the
latitude: "41.842258"        # decimal latitude of workshop venue (use https://www.latlong.net/)
longitude: "-88.245781"       # decimal longitude of the workshop venue (use https://www.latlong.net)
humandate: "Synchronously January, 2023"    # human-readable dates for the workshop (e.g., "Feb 17-18, 2020")
humantime: "24/7"    # human-readable times for the workshop (e.g., "9:00 am - 4:30 pm")
startdate: 2023-01-09 # machine-readable start date for the workshop in YYYY-MM-DD format like 2015-01-01
enddate: 2023-01-13   # machine-readable end date for the workshop in YYYY-MM-DD format like 2015-01-02
email: ["brunella.d'anzi@cern.ch"]
---

![CMS Detector Slice](https://cmsexperiment.web.cern.ch/sites/cmsexperiment.web.cern.ch/files/detectoroverview.gif){:width="50%"}

## Welcome to the CMS Data Analysis School 2023 Tracking and Vertexing Short Exercise!

We will present an introduction to using tracks for analyses in the era of **large pile-up** (many primary vertices). Our exercises will all use real data and will familiarize you with the following techniques:

> ## Exercise Goals
> - Extracting **basic track parameters** and reconstructing invariant masses from tracks in CMSSW. Tracks are the detector entities that are closest to the four-vectors of particles: the momentum of a track is nearly the momentum of the charged particle itself.
> - Cleaning **sets of tracks for analysis**. We will use filters to eliminate bad tracks and discuss sources of tracking uncertainties. These filters are provided by the tracking POG (Physics Object Group)
> - Extracting **basic parameters of primary vertices**. In this high-luminosity era, it is not uncommon for a single event to contain as many as thirty to fifty independent collisions. For most analyses, only one is relevant, and it can usually be identified by its tracks.
> - Using tracks to measure from data and Monte Carlo (MC) samples the **tracking efficiency** by using tag and probe method via dimuon resonances. It can be extended to measure the efficiency of any selection of muons (trigger, isolation, identification) and derive scale factors from MC.
{: .checklist}

> ## Prerequisites
> **Before going any further, please complete the [CMS DAS Pre-Exercises](https://fnallpc.github.io/cms-das-pre-exercises/) and then follow the instructions on the [setup page](setup.md).**
{: .prereq}

> ## Facilitators
> * [Brunella D'Anzi*](https://twiki.cern.ch/twiki/bin/view/Main/BrunellaDAnzi), University and INFN of Bari ([brunella.d'anzi@cern.ch](mailto:brunella.d'anzi@cern.ch)) 
> * [Caleb Smith](https://twiki.cern.ch/twiki/bin/view/Main/CalebJamesSmith), Baylor ([caleb.smith@cern.ch](mailto:caleb.smith@cern.ch)) 
> * [Nicola De Filippis](https://twiki.cern.ch/twiki/bin/view/Main/NicolaDeFilippis), INFN and Polytechnic of Bari ([nicola.defilippis@ba.infn.it](mailto:nicola.defilippis@ba.infn.it)) 
> * [Susan J Dittmer](https://twiki.cern.ch/twiki/bin/view/Main/SusanDittmer), University of Illinois Chicago ([Susan.Dittmer@cern.ch](mailto:susan.dittmer@cern.ch))
>  
> *Lead Contact
> <table>
>   <tr>
>     <td align="center"><a href="https://github.com/bdanzi"><img src="https://avatars.githubusercontent.com/u/75045014?s=96&v=4" width="100px;" alt=""/><br /><sub><b>Brunella D'Anzi</b></sub></a><br /><a href="https://web2.ba.infn.it/bdanzi//" title="More about her">🖋</a></td>
>     <td align="center"><a href="https://physics.ku.edu/people/smith-caleb"><img src="https://lpc.fnal.gov/CMSDAS2022/Caleb_Smith.jpg" width="85px;" alt=""/><br /><sub><b>Caleb Smith</b></sub></a><br /><a href="https://physics.ku.edu/people/smith-caleb" title="More about him">🖋</a></td>
>     <td align="center"><a href="https://lpc.fnal.gov/fellows/2018/Nicola_De_Filippis.shtml"><img src="https://lpc.fnal.gov/fellows/imgs/med/2018/Nicola_De_Filippis.jpg" width="100px;" alt=""/><br /><sub><b>Nicola De Filippis</b></sub></a><br /><a href="https://lpc.fnal.gov/fellows/2018/Nicola_De_Filippis.shtml" title="More about him">🖋</a></td>
>     <td align="center"><a href="https://hep.phys.uic.edu/post-docs/"><img src="https://lpc.fnal.gov/CMSDAS2022/No_Photo.png" width="100px;" alt=""/><br /><sub><b>Susan J Dittmer</b></sub></a><br /><a href="https://hep.phys.uic.edu/post-docs/" title="More about her">🖋</a></td>
>   </tr>
> </table>
{: .testimonial}

> ## Mattermost Chat
> **The [Tracking and Vertexing Short Exercise](https://mattermost.web.cern.ch/cmsdaslpc2023/channels/shortextrackingvertexing) channel will be available once you join the [CMSDAS@LPC2023](https://mattermost.web.cern.ch/cmsdaslpc2023/channels/town-square) team. Direction for how to join this Mattermost chat team can be found on the <a href="setup.html">setup</a> page.**
{: .discussion}

> ## CERN Twiki and Introduction Slides
> **The CERN TWiki of this short exercise can be found at [this link](https://twiki.cern.ch/twiki/bin/view/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise). At the beginning of this short exercise an introduction will be done based on [these slides](https://docs.google.com/viewer?url=https://raw.githubusercontent.com/bdanzi/trackingvertexing/gh-pages/files/CMSDAS2023_TrackingVertexingExercise_Introduction.pdf).**
{: .callout}
{% comment %} This is a comment in Liquid {% endcomment %}

> ## Closeout Slides
> **You can find a guideline closeup for the Tracking and Vertexing Short Exercise questions and results on [these slides](https://docs.google.com/viewer?url=https://raw.githubusercontent.com/bdanzi/trackingvertexing/gh-pages/files/CMSDAS2023_TrackingVertexingExercise_Wrapup.pdf).**
{: .challenge}
{% comment %} This is a comment in Liquid {% endcomment %}

{% include links.md %}
