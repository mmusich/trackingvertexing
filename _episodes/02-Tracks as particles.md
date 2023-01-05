---
title: "Tracks as particles"
teaching: 10
exercises: 10
questions:
- "Can we consider a track as a particle?"
- "Can I use an alternative to the muon object?"
- "Can I define the invariant mass of two tracks?"
objectives:
- "Being familiar with the tracks, particle and identification concepts."
- "Plot distributions of variables related to a muon resonance."
keypoints:
- "Tracks give us a direct handle on actual particles, thus they can easily be used to reconstruct other particles in the event"

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
<a href="https://raw.githubusercontent.com/bdanzi/trackingvertexing/gh-pages/data/cms_quarterview.png"><img src = "https://raw.githubusercontent.com/bdanzi/trackingvertexing/gh-pages/data/cms_quarterview.png" alt="CMS Quarter-view." width ="500"></a>
To increase the chances that pairs of randomly chosen tracks are descendants of the same decay, consider a smaller set of tracks: muons. Muons are identified by the fact that they can pass through meters of iron (the CMS magnet return yoke), so muon tracks extend from the silicon tracker to the muon chambers (see CMS quarter-view below), as much as 12 meters long! Muons are rare in hadron collisions. If an event contains two muons, they often (though not always) come from the same decay.

Normally, one would access muons through the `reco::Muon` object since this contains additional information about the quality of the muon hypothesis. For simplicity, we will access their track collection in the same way that we have been accessing the main track collection. We only need to replace `generalTracks` with `globalMuons`. Add the following loop to `kinematics.py`.
~~~
events.toBegin()
for i, event in enumerate(events):
    if i >= 15: break            # only the first 15 events
    print "Event", i
    event.getByLabel("globalMuons", tracks)
    for j, track in enumerate(tracks.product()):
        print "    Track", j, track.charge()/track.pt(), track.phi(), track.eta(), track.dxy(), track.dz()
~~~
{: .language-python}
Run this code on the `run321167_Charmonium_AOD.root` file that you can copy with:
~~~
xrdcp root://cmseos.fnal.gov//store/user/cmsdas/2023/short_exercises/trackingvertexing/run321167_Charmonium_AOD.root .
~~~
{: .language-bash}
Notice how few muon tracks there are compared to the same code executed for `generalTracks`. In fact, you only see as many muons as you do because this data sample was collected with a muon trigger. (The muon definition in the trigger is looser than the `globalMuons` algorithm, which is why there are some events with fewer than two `globalMuons`.)
See in the `Appendix` an application for the Muon and Tracks objects usage in the CMS tracking efficiency computation.

As an exercise, make a histogram of all di-muon masses from 0 to 5 [GeV](https://twiki.cern.ch/twiki/bin/view/CMS/GeV)). Exclude events that do not have exactly two muon tracks, and note that the muon mass is 0.106 [GeV](https://twiki.cern.ch/twiki/bin/view/CMS/GeV)). Create a file `dimuon_mass.py` in `TrackingShortExercize/` for this purpose.

> ## More...
> The solution combines several of the techniques introduced above:
> ~~~
> import math
> import DataFormats.FWLite as fwlite
> import ROOT
> 
> events = fwlite.Events("file:run321167_Charmonium_AOD.root")
> tracks = fwlite.Handle("std::vector<reco::Track>")
> mass_histogram = ROOT.TH1F("mass", "mass", 100, 0.0, 5.0)
> 
> events.toBegin()
> for event in events:
>     event.getByLabel("globalMuons", tracks)
>     product = tracks.product()
>     if product.size() == 2:
>         one = product[0]
>         two = product[1]
>         if not (one.charge()*two.charge() == -1):  continue
>         energy = (math.sqrt(0.106**2 + one.p()**2) +
>                   math.sqrt(0.106**2 + two.p()**2))
>         px = one.px() + two.px()
>         py = one.py() + two.py()
>         pz = one.pz() + two.pz()
>         mass = math.sqrt(energy**2 - px**2 - py**2 - pz**2)
>         mass_histogram.Fill(mass)
> 
> c = ROOT.TCanvas ("c", "c", 800, 800)
> mass_histogram.Draw()
> c.SaveAs("mass.png")
> ~~~
> {: .language-python}
> The histogram should look like this:
> 
> If so, congratulations! You've discovered the J/ψ!
> <a href="https://raw.githubusercontent.com/bdanzi/trackingvertexing/gh-pages/data/mass.png"><img src = "https://raw.githubusercontent.com/bdanzi/trackingvertexing/gh-pages/data/mass.png" alt="Invariant Mass of the J/Psi" width ="500"></a>
{: .solution}
{% include links.md %}

