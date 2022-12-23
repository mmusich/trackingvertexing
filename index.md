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
startdate: 2023-01-09      # machine-readable start date for the workshop in YYYY-MM-DD format like 2015-01-01
enddate: 2023-01-13        # machine-readable end date for the workshop in YYYY-MM-DD format like 2015-01-02
email: ["brunella.d'anzi@cern.ch"]
#email: ["gabriele.benelli@cern.ch", "tamer.elkafrawy@cern.ch", "pampa.ghose@cern.ch", "rmh16d@my.fsu.edu", "cmadrid@fnal.gov", "ch.mclean@cern.ch", andrew.malone.melo@cern.ch", "stephen.mrenna@cern.ch", "alexx.stephen.perloff@cern.ch", "marguerite.belt.tonjes@cern.ch", "hannsjorg.artur.weber@cern.ch", "jingyu.zhang@cern.ch"]    # boxed, comma-separated list of contact email addresses for the host, lead instructor, or whoever else is handling questions, like ["marlyn.wescoff@example.org", "fran.bilas@example.org", "ruth.lichterman@example.org"]
---

![CMS Detector Slice](https://cmsexperiment.web.cern.ch/sites/cmsexperiment.web.cern.ch/files/detectoroverview.gif){:width="50%"}

We will present an introduction to using tracks for analyses in the era of large pile-up (many primary vertices). Our exercises will all use real data and will familiarize you with the following techniques:

- Extracting basic track parameters and reconstructing invariant masses from tracks in CMSSW. Tracks are the detector entities that are closest to the four-vectors of particles: the momentum of a track is nearly the momentum of the charged particle itself.
- Cleaning sets of tracks for analysis. We will use filters to eliminate bad tracks and discuss sources of tracking uncertainties. These filters are provided by the tracking POG (Physics Object Group)
- Extracting basic parameters of primary vertices. In this high-luminosity era, it is not uncommon for a single event to contain as many as thirty to fifty independent collisions. For most analyses, only one is relevant, and it can usually be identified by its tracks.
- Using tracks to measure from data and MC the muon tracking efficiency by using tag and probe method. It can be extended to measure the efficiency of any selection of muons (trigger, isolation, identification) and derive scale factors from MC.
<!-- this is an html comment -->

> ## Facilitators
> [Brunella D'Anzi](https://twiki.cern.ch/twiki/bin/view/Main/BrunellaDAnzi) (University of Bari), [Caleb Smith](https://twiki.cern.ch/twiki/bin/view/Main/CalebJamesSmith) (Baylor), [Nicola De Filippis](https://twiki.cern.ch/twiki/bin/view/Main/NicolaDeFilippis) (Polytechnic of Bari), [Susan Dittmer](https://twiki.cern.ch/twiki/bin/view/Main/SusanDittmer) (University of Illinois Chicago)
{: .testimonial}

> ## Introduction Slides
> At the beginning of this short exercise an introduction will be done based on [these slides](https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/CMSDAS2023_TrackingVertexingExercise_Introduction.pdf)
{: .callout}
{% comment %} This is a comment in Liquid {% endcomment %}

> ## Prerequisites
> **Before going any further, please follow the instructions on the [setup page](setup.md).**
> [CMS DAS Pre-Exercises](https://fnallpc.github.io/cms-das-pre-exercises/)
{: .prereq}

{% include links.md %}
