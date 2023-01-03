---
title: "Tracks as particles"
teaching: 0
exercises: 30
questions:
- ""
objectives:
- ""
- ""
- "" 
keypoints:
- "It is extremely important to be sinchronized with the environment used to start working as a team!"
- "Managing ROOT files in High Energy Physics should be the first thing to be familiar with."
---
Unlike calorimeter showers, tracks can usually be interpreted as particle 4-vectors without any additional corrections. Detector alignment, non-helical trajectories from energy loss, Lorentz angle corrections, and (to a much smaller extent) magnetic field inhomogeneities are important, but they are all corrections that must be applied during or before the track-reconstruction process. From an analyzer's point of view, most tracks are individual particles (depending on quality cuts) and the origin and momentum of the particle are derived from the track's geometry, with some resolution (random error). Biases (systematic offsets from the true values) are not normal: they're an indication that something went wrong in this process.
The analyzer does not even need to calculate the particle's momentum from the track parameters: there are member functions for that. Particle's transverse momentum, momentum magnitude, and all of its components can be read through the following lines (let's name this new file kinematics.py and create it in `TrackingShortExercize/`):
~~~
import DataFormats.FWLite as fwlite
import ROOT

events = fwlite.Events("file:run321167_ZeroBias_AOD.root")
tracks = fwlite.Handle("std::vector<reco::Track>")

for i, event in enumerate(events):
    event.getByLabel("generalTracks", tracks)
    for track in tracks.product():
        print track.pt(), track.p(), track.px(), track.py(), track.pz()
    if i > 20: break
~~~
{: .language-python}
Now we can use this to do some kinematics. Assuming that the particle is a pion (pion mass = 0.140 [GeV](https://twiki.cern.ch/twiki/bin/view/CMS/GeV)), calculate its kinetic energy.
> ## Answer
> ~~~
> import DataFormats.FWLite as fwlite
> import ROOT
> import math
> 
> events = fwlite.Events("file:run321167_ZeroBias_AOD.root")
> tracks = fwlite.Handle("std::vector<reco::Track>")
> 
> for i, event in enumerate(events):
>     event.getByLabel("generalTracks", tracks)
>     for track in tracks.product():
>         print track.pt(), track.p(), track.px(), track.py(), track.pz()
>         print "energy: ", math.sqrt(0.140**2 + track.p()**2) - 0.140
>     if i > 20: break
> ~~~
> {: .language-python}
{: .solution}
{% include links.md %}

