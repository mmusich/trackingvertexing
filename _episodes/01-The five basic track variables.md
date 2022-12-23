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

## Accessing track variables

Create `print.py` (for example `emacs -nw print.py`, or use your favorite text editor) in TrackingShortExercize, then copy-paste the following code and run it (`python print.py`). Please note, if your `run321457_ZeroBias_AOD.root` is not in the directory you're working from, be sure to use the appropriate path in line 2.
~~~
import DataFormats.FWLite as fwlite
events = fwlite.Events("file:run321167_ZeroBias_AOD.root")
tracks = fwlite.Handle("std::vector<reco::Track>")

for i, event in enumerate(events):
    if i >= 5: break # print info only about the first 5 events
    print "Event", i
    event.getByLabel("generalTracks", tracks)
    for j, track in enumerate(tracks.product()):
        print "    Track", j,
        print "\t charge/pT: %.3f" %(track.charge()/track.pt()),
        print "\t phi: %.3f" %track.phi(),
        print "\t eta: %.3f" %track.eta(),
        print "\t dxy: %.4f" %track.dxy(),
        print "\t dz: %.4f"  %track.dz()

~~~
{: .language-python}

The first three lines load the `FWLite framework`, the data file, and prepare a handle for the track collection using its full C++ name (`std::vector`). In each event, we load the tracks labeled `generalTracks` and loop over them, printing out the five basic track variables for each. The C++ equivalent of this is hidden below (longer and more difficult to write and compile, but faster to execute on large datasets) and is optional for this entire short exercise.

> ## C++ version
> ~~~
> cd ${CMSSW_BASE}/src
> mkdir MyDirectory
> cd MyDirectory
> mkedanlzr PrintOutTracks
> cd ..
> ~~~
> {: .language-bash}
> Use your favorite editor to add (if missing) the following line at the top of `MyDirectory/PrintOutTracks/plugins/BuildFile.xml`
> (e.g. `emacs -nw MyDirectory /PrintOutTracks/plugins/BuildFile.xml`):
> ~~~
> <use name="DataFormats/TrackReco"/>
> ~~~
> {: .language-*}
> Now edit `MyDirectory/PrintOutTracks/plugins/PrintOutTracks.cc` and put (if missing) the following in the `#include` section:
> ~~~
> #include <iostream>
> #include "DataFormats/TrackReco/interface/Track.h"
> #include "DataFormats/TrackReco/interface/TrackFwd.h"
> #include "FWCore/Utilities/interface/InputTag.h"
> ~~~
> {: .language-*}
> Inside the `PrintOutTracks` class definition (one line below the member data comment, before the }), replace `edm::EDGetTokenT<TrackCollection> tracksToken_;` with:
> ~~~
> edm::EDGetTokenT<edm::View<reco::Track> > tracksToken_;  //used to select which tracks to read from configuration file
> int indexEvent_;
> ~~~
> {: .language-*}
> Inside the `PrintOutTracks::PrintOutTracks(const edm::ParameterSet& iConfig)` constructor before the "{", modify the consumes statement to read:
> ~~~
> : 
> tracksToken_(consumes<edm::View<reco::Track> >(iConfig.getUntrackedParameter<edm::InputTag>("tracks", > edm::InputTag("generalTracks")) ))
> ~~~
> {: .language-*}
{: .solution}
{% include links.md %}

