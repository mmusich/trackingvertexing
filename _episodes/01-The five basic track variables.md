---
title: "The five basic track variables"
teaching: 0
exercises: 30
questions:
- "What are the CMS track variables?"
objectives:
- "Being familiar with tracks collections and their variables."
keypoints:
- "It is extremely important to be sinchronized with the environment used to start working as a team!"
- "Managing ROOT files in High Energy Physics should be the first thing to be familiar with."
---
One of the oldest tricks in particle physics is to put a track-measuring device in a strong, roughly uniform magnetic field so that the tracks curve with a radius proportional to their momenta (see [derivation](http://en.wikipedia.org/wiki/Gyroradius#Relativistic_case)). Apart from energy loss and magnetic field inhomogeneities, the particles' trajectories are helices. This allows us to measure a dynamic property (momentum) from a geometric property (radius of curvature).<a href="http://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Helix.svg/410px-Helix.svg.png"><img src = "http://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Helix.svg/410px-Helix.svg.png" alt="Helical trajectory of a particle in a vertical magnetic field." width ="200"></a>

A helical trajectory can be expressed by five parameters, but the parameterization is not unique. Given one parameterization, we can always re-express the same trajectory in another parameterization. Many of the data fields in a CMSSW reco::Track are alternate ways of expressing the same thing, and there are functions for changing the reference point from which the parameters are expressed. (For a much more detailed description, see [this page](http://www-jlc.kek.jp/subg/offl/lib/docs/helix_manip/node3.html#SECTION00210000000000000000).)<a href="https://www.lhc-closer.es/webapp/files/1435504163_ad6fd1cc4163a3a2d3c54388c80c45ba.jpg"><img src = "https://www.lhc-closer.es/webapp/files/1435504163_ad6fd1cc4163a3a2d3c54388c80c45ba.jpg" alt="Track parameters at LHC" width ="200"/></a>

In general terms, the five parameters are:

•	signed radius of curvature (units of cm), which is proportional to particle charge divided by the transverse momentum, pT (units of GeV);

•	angle of the trajectory at a given point on the helix, in the plane transverse to the beamline (usually called φ);

•	angle of the trajectory at a given point on the helix with respect to the beamline (θ, or equivalently λ = π/2 - θ), which is usually expressed in terms of [pseudorapidity](https://en.wikipedia.org/wiki/Pseudorapidity) (η = −ln(tan(θ/2)));

•	offset or " impact parameter " relative to some reference point (usually the beamspot or a selected primary vertex), in the plane transverse to the beamline (usually called dxy);

•	impact parameter relative to a reference point (beamspot or a selected primary vertex), along the beamline (usually called dz).

The exact definitions are given in the `reco::TrackBase` [header file](https://github.com/cms-sw/cmssw/blob/CMSSW_10_2_7/DataFormats/TrackReco/interface/TrackBase.h). This is also where most tracking variables and functions are defined. The rest are in the `reco::Track` [header file](https://github.com/cms-sw/cmssw/blob/CMSSW_10_2_7/DataFormats/TrackReco/interface/TrackBase.h), but most data fields in the latter are accessible only in RECO (full data record), not AOD/MINIAOD/NANOAOD (the subsets that are available to physics analyses).
~~~
print("Hello World")
~~~
{: .language-python}

{% include links.md %}

