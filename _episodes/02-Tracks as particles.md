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
> ## Additional information
> Identifying the particle that made the track is difficult: the mass of some low-momentum tracks can be identified by their energy loss, called dE/dx, and electrons and muons can be identified by signatures in other subdetectors. Without any other information, the safest assumption is that a randomly chosen track is a pion, since hadron collisions produce a lot of pions.
{: .solution}
Let's look for resonances. Given two tracks,
~~~
if len(tracks.product()) > 1:

one = tracks.product()[0]
two = tracks.product()[1]
~~~
{: .language-python}
the invariant mass may be calculated as
~~~
total_energy = math.sqrt(0.140**2 + one.p()**2) + math.sqrt(0.140**2 + two.p()**2)
total_px = one.px() + two.px()
total_py = one.py() + two.py()
total_pz = one.pz() + two.pz()
mass = math.sqrt(total_energy**2 - total_px**2 - total_py**2 - total_pz**2)
~~~
{: .language-python}
However, this quantity has no meaning unless the two particles are actually descendants of the same decay. Two randomly chosen tracks (**out of hundreds per event**) typically are not.
{% include links.md %}

