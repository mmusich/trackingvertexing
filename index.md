---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
---

![CMS Detector Slice](https://cmsexperiment.web.cern.ch/sites/cmsexperiment.web.cern.ch/files/detectoroverview.gif){:width="50%"}

We will present an introduction to using tracks for analyses in the era of large pile-up (many primary vertices). Our exercises will all use real data and will familiarize you with the following techniques:

- Extracting basic track parameters and reconstructing invariant masses from tracks in CMSSW. Tracks are the detector entities that are closest to the four-vectors of particles: the momentum of a track is nearly the momentum of the charged particle itself.
- Cleaning sets of tracks for analysis. We will use filters to eliminate bad tracks and discuss sources of tracking uncertainties. These filters are provided by the tracking POG (Physics Object Group)
- Extracting basic parameters of primary vertices. In this high-luminosity era, it is not uncommon for a single event to contain as many as thirty to fifty independent collisions. For most analyses, only one is relevant, and it can usually be identified by its tracks.
- Using tracks to measure from data and MC the muon tracking efficiency by using tag and probe method. It can be extended to measure the efficiency of any selection of muons (trigger, isolation, identification) and derive scale factors from MC.
<!-- this is an html comment -->

{% comment %} This is a comment in Liquid {% endcomment %}

> ## Prerequisites
>
> [CMS DAS Pre-Exercises](https://fnallpc.github.io/cms-das-pre-exercises/)
{: .prereq}

{% include links.md %}
