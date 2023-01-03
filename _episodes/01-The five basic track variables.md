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
> {: .language-cpp}
> Now edit `MyDirectory/PrintOutTracks/plugins/PrintOutTracks.cc` and put (if missing) the following in the `#include` section:
> ~~~
> #include <iostream>
> #include "DataFormats/TrackReco/interface/Track.h"
> #include "DataFormats/TrackReco/interface/TrackFwd.h"
> #include "FWCore/Utilities/interface/InputTag.h"
> ~~~
> {: .language-cpp}
> Inside the `PrintOutTracks` class definition (one line below the member data comment, before the }), replace `edm::EDGetTokenT<TrackCollection> tracksToken_;` with:
> ~~~
> edm::EDGetTokenT<edm::View<reco::Track> > tracksToken_;  //used to select which tracks to read from configuration file
> int indexEvent_;
> ~~~
> {: .language-cpp}
> Inside the `PrintOutTracks::PrintOutTracks(const edm::ParameterSet& iConfig)` constructor before the "{", modify the consumes statement to read:
> ~~~
> : 
> tracksToken_(consumes<edm::View<reco::Track> >(iConfig.getUntrackedParameter<edm::InputTag>("tracks", edm::InputTag("generalTracks")) ))
> ~~~
> {: .language-cpp}
> after the "//now do what ever initialization is needed" comment:
> ~~~
> indexEvent_ = 0;
> ~~~
> {: .language-cpp}
> Put the following inside the `PrintOutTracks::analyze` method:
> ~~~
>   std::cout << "Event " << indexEvent_ << std::endl;
>
>   edm::Handle<edm::View<reco::Track> > trackHandle;
>   iEvent.getByToken(tracksToken_, trackHandle);
>   if ( !trackHandle.isValid() ) return;
>   const edm::View<reco::Track>& tracks = *trackHandle;
>   size_t iTrack = 0;
>   for ( auto track : tracks ) {
>     std::cout << "    Track " << iTrack << " "
>        << track.charge()/track.pt() << " "
>               << track.phi() << " "
>               << track.eta() << " "
>               << track.dxy() << " "
>               << track.dz()
>        << std::endl;
>     iTrack++;
>   }
>   ++indexEvent_;
> ~~~
> {: .language-cpp}
> Now compile it by running:
> ~~~
> scram build -j 4
> ~~~
> {: .language-bash}
> Go back to `TrackingShortExercize/` and create a CMSSW configuration file named `run_cfg.py` (`emacs -nw run_cfg.py`):
> ~~~
> import FWCore.ParameterSet.Config as cms
> 
> process = cms.Process("RUN")
> 
> process.source = cms.Source("PoolSource",
>     fileNames = cms.untracked.vstring("file:run321167_ZeroBias_AOD.root"))
> process.maxEvents = cms.untracked.PSet(input = cms.untracked.int32(5))
> 
> process.MessageLogger = cms.Service("MessageLogger",
>     destinations = cms.untracked.vstring("cout"),
>     cout = cms.untracked.PSet(threshold = cms.untracked.string("ERROR")))
> 
> process.PrintOutTracks = cms.EDAnalyzer("PrintOutTracks")
> process.PrintOutTracks.tracks = cms.untracked.InputTag("generalTracks")
> 
> process.path = cms.Path(process.PrintOutTracks)
> ~~~
> {: .language-python}
> And, finally, run it with:
> ~~~
> cmsRun run_cfg.py
> ~~~
> {: .language-bash}
> This will produce the same output as the Python script, but it can be used on huge datasets. Though the language is different, notice that C++ and FWLite use the same names for member functions: `charge()`, `pt()`, `phi()`, `eta()`, `dxy()`, and `dz()`.
> That is intentional: you can learn what kinds of data are available with interactive FWLite and then use the same access methods when writing GRID jobs. There is another way to access `FWLite` with ROOT's C++-like syntax.
> The plugin is here: `/eos/uscms/store/user/cmsdas/2023/short_exercises/trackingvertexing/MyDirectory/PrintOutTracks/plugins/PrintOutTracks.cc`
> The run_cfg.py is here: `/eos/uscms/store/user/cmsdas/2023/short_exercises/trackingvertexing/run_cfg.py`
{: .solution}

## Track quality variables

The first thing you should notice is that each event has hundreds of tracks. That is because hadronic collisions produce large numbers of particles and `generalTracks` is the broadest collection of tracks identified by CMSSW reconstruction. Some of these tracks are not real (ghosts, duplicates, noise...) and a good analysis should define quality cuts to select tracks requiring a certain quality.
Some analyses remove spurious tracks by requiring them to come from the beamspot (small dxy, dz). Some require high-momentum (usually high transverse momentum, pT), but that would be a bad idea in a search for decays with a small mass difference such as ψ' → J/ψ π+π−. In general, each analysis group should review their own needs and ask the Tracking POG about standard selections.
Some of these standard selections have been encoded into a quality flag with three categories: `loose`, `tight`, and `highPurity`. All tracks delivered to the analyzers are at least `loose`, `tight` is a subset of these that are more likely to be real, and "highPurity" is a subset of "tight" with even stricter requirements. There is a trade-off: `loose` tracks have high efficiency but also high backgrounds, `highPurity` has slightly lower efficiency but much lower backgrounds, and `tight` is in between (see also the plots below). As of `CMSSW 7.4`, these are all calculated using MVAs (**M**ulti**V**ariate **A**nalysis techniques) for the various iterations. In addition to the status bits, it's also possible to access the MVA values directly.

Plot of Efficiency vs Eta:
<a href="https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_eta.png"><img src = "https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_eta.png" alt="Efficiency vs Eta" width ="350"></a>
Plot of Efficiency vs pT:
<a href="https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png"><img src = "https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png" alt="Efficiency vs pT" width ="350"></a>
Plot of Backgrounds vs Eta:
<a href="https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_fakerate_eta.png"><img src = "https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_fakerate_eta.png" alt="Backgrounds vs Eta" width ="350"></a>
Plot of Backgrounds vs pT:
<a href="https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_fakerate_pt.png"><img src = "https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_fakerate_pt.png" alt="Backgrounds vs pT" width ="350"></a>

Update the file `print.py` with the following lines:
Add a `Handle` to the MVA values:
~~~
MVAs = fwlite.Handle("std::vector<float>")
~~~
{: .language-python}
The event loop should be updated to this:
~~~
for i, event in enumerate(events):
    if i >= 5: break            # only the first 5 events
    print "Event", i
    event.getByLabel("generalTracks", tracks)
    event.getByLabel("generalTracks", "MVAValues", MVAs)

    numTotal = tracks.product().size()
    if numTotal == 0: continue
    numLoose = 0
    numTight = 0
    numHighPurity = 0

    for j, (track, mva) in enumerate(zip(tracks.product(), MVAs.product())):
        if track.quality(track.qualityByName("loose")):      numLoose      += 1
        if track.quality(track.qualityByName("tight")):      numTight      += 1
        if track.quality(track.qualityByName("highPurity")): numHighPurity += 1

        print "    Track", j,
        print "\t charge/pT: %.3f" %(track.charge()/track.pt()),
        print "\t phi: %.3f" %track.phi(),
        print "\t eta: %.3f" %track.eta(),
        print "\t dxy: %.4f" %track.dxy(),
        print "\t dz: %.4f"  %track.dz(),
        print "\t nHits: %s" %track.numberOfValidHits(), "(%s P+ %s S)"%(track.hitPattern().numberOfValidPixelHits(),track.hitPattern().numberOfValidStripHits()),
        print "\t algo: %s"  %track.algoName(),
        print "\t mva: %.3f" %mva

    print "Event", i,   
    print "numTotal:", numTotal,
    print "numLoose:", numLoose, "(%.1f %%)"%(float(numLoose)/numTotal*100),
    print "numTight:", numTight, "(%.1f %%)"%(float(numTight)/numTotal*100),  
    print "numHighPurity:", numHighPurity, "(%.1f %%)"%(float(numHighPurity)/numTotal*100)
~~~
{: .language-python}
The C++-equivalent is hidden below.
> ## C++ version
> Update your `EDAnalyzer` adding the following lines:
>
> To class declaration:
> ~~~
> edm::EDGetTokenT<edm::View<float> > mvaValsToken_;
> ~~~
> {: .language-cpp}
> To constructor:
> ~~~
> , mvaValsToken_( consumes<edm::View<float> >(iConfig.getUntrackedParameter<edm::InputTag>("mvaValues", edm::InputTag("generalTracks", "MVAValues")) ) )
> ~~~
> {: .language-cpp}
> Change your `analyze` method to:
> ~~~
>   std::cout << "Event " << indexEvent_ << std::endl;
>
>   edm::Handle<edm::View<reco::Track> > trackHandle;
>   iEvent.getByToken(tracksToken_, trackHandle);
>   if ( !trackHandle.isValid() ) return;
>   const auto numTotal = trackHandle->size();
>
>   edm::Handle<edm::View<float> > trackMVAstoreHandle;
>   iEvent.getByToken(mvaValsToken_,trackMVAstoreHandle);
>   if ( !trackMVAstoreHandle.isValid() ) return;
>
>   auto numLoose = 0;
>   auto numTight = 0;
>   auto numHighPurity = 0;
>
>   const edm::View<reco::Track>& tracks = *trackHandle;
>   size_t iTrack = 0;
>   for ( auto track : tracks ) {
>     if (track.quality(track.qualityByName("loose"))     ) ++numLoose;
>     if (track.quality(track.qualityByName("tight"))     ) ++numTight;
>     if (track.quality(track.qualityByName("highPurity"))) ++numHighPurity;
>
>     std::cout << "    Track " << iTrack << " "
>        << track.charge()/track.pt() << " "
>               << track.phi() << " "
>               << track.eta() << " "
>               << track.dxy() << " "
>               << track.dz()
>               << track.chi2() << " "
>        << track.ndof() << " "
>               << track.numberOfValidHits() << " "
>               << track.algoName() << " "
>               << trackMVAstoreHandle->at(iTrack)
>               << std::endl;
>     iTrack++;
>   }
>   ++indexEvent_;
>
>   std::cout << "Event " << indexEvent_
>             << " numTotal: " << numTotal
>             << " numLoose: " << numLoose
>             << " numTight: " << numTight
>             << " numHighPurity: " << numHighPurity
>             << std::endl;
>
> ~~~
> {: .language-cpp}
> Go back to `TrackingShortExercize/` and edit the CMSSW configuration file named `run_cfg.py` (`emacs -nw run_cfg.py`), adding the following line:
> ~~~
> process.PrintOutTracks.mvaValues = cms.untracked.InputTag("generalTracks","MVAValues")
> ~~~
> {: .language-python}
> Now run the `analyzer`:
> ~~~
> cmsRun run_cfg.py
> ~~~
> {: .language-python}
> The actual `plugin` I used can be found in `/eos/uscms/store/user/cmsdas/2023/short_exercises/trackingvertexing/PrintOutTracks_MVA.cc`
> The actual `CMSSW config` file can be found in `/eos/uscms/store/user/cmsdas/2023/short_exercises/trackingvertexing/run_cfg_MVA.py`
{: .solution}
Now prepare plots for the track variables discussed above, as in the example below (name this file `plot_track_quantities.py` and put it in `TrackingShortExercize/`). Compare the distributions of track-quality-related variables (number of pixel hits, track goodness of fit, ...) between tracks passing the `highPurity` and `Loose` quality flags.
> ## Answer
> ~~~
> import DataFormats.FWLite as fwlite
> import ROOT
> 
> events = fwlite.Events("run321167_ZeroBias_AOD.root")
> tracks = fwlite.Handle("std::vector<reco::Track>")
> 
> hist_pt       = ROOT.TH1F("pt",       "track pt; p_{T} [GeV]", 100, 0.0, 100.0)
> hist_eta      = ROOT.TH1F("eta",      "track eta; #eta", 60, -3.0, 3.0)
> hist_phi      = ROOT.TH1F("phi",      "track phi; #phi", 64, -3.2, 3.2)
> 
> hist_loose_normChi2     = ROOT.TH1F("hist_loose_normChi2"    , "norm. chi2; norm #chi^{2}"        , 100, 0.0, 10.0)
> hist_loose_numPixelHits = ROOT.TH1F("hist_loose_numPixelHits", "pixel hits; # pixel hits"         , 15, 0.0, 15.0)
> hist_loose_numValidHits = ROOT.TH1F("hist_loose_numValidHits", "valid hits; # valid hits"         , 35, 0.0, 35.0)
> hist_loose_numTkLayers  = ROOT.TH1F("hist_loose_numTkLayers" , "valid layers; # valid Tk layers"  , 25, 0.0, 25.0)
> hist_highP_normChi2     = ROOT.TH1F("hist_highP_normChi2"    , "norm. chi2; norm #chi^{2}"        , 100, 0.0, 10.0)
> hist_highP_numPixelHits = ROOT.TH1F("hist_highP_numPixelHits", "pixel hits; # pixel hits"         , 15, 0.0, 15.0)
> hist_highP_numValidHits = ROOT.TH1F("hist_highP_numValidHits", "valid hits; # valid hits"         , 35, 0.0, 35.0)
> hist_highP_numTkLayers  = ROOT.TH1F("hist_highP_numTkLayers" , "valid layers; # valid Tk layers"  , 25, 0.0, 25.0)
> 
> hist_highP_normChi2.SetLineColor(ROOT.kRed)
> hist_highP_numPixelHits.SetLineColor(ROOT.kRed)
> hist_highP_numValidHits.SetLineColor(ROOT.kRed)
> hist_highP_numTkLayers.SetLineColor(ROOT.kRed)
> 
> for i, event in enumerate(events):
>     event.getByLabel("generalTracks", tracks)
>     for track in tracks.product():
>         hist_pt.Fill(track.pt())
>         hist_eta.Fill(track.eta())
>         hist_phi.Fill(track.phi())
> 
>         if track.quality(track.qualityByName("loose")):
>             hist_loose_normChi2    .Fill(track.normalizedChi2())
>             hist_loose_numPixelHits.Fill(track.hitPattern().numberOfValidPixelHits())
>             hist_loose_numValidHits.Fill(track.hitPattern().numberOfValidHits())
>             hist_loose_numTkLayers .Fill(track.hitPattern().trackerLayersWithMeasurement())
>         if track.quality(track.qualityByName("highPurity")):
>             hist_highP_normChi2    .Fill(track.normalizedChi2())
>             hist_highP_numPixelHits.Fill(track.hitPattern().numberOfValidPixelHits())
>             hist_highP_numValidHits.Fill(track.hitPattern().numberOfValidHits())
>             hist_highP_numTkLayers .Fill(track.hitPattern().trackerLayersWithMeasurement())
> 
>     if i > 500: break
> 
> c = ROOT.TCanvas( "c", "c", 800, 800)
> 
> hist_pt.Draw()
> c.SetLogy()
> c.SaveAs("track_pt.png")
> c.SetLogy(False)
> 
> hist_eta.Draw()
> c.SaveAs("track_eta.png")
> hist_phi.Draw()
> c.SaveAs("track_phi.png")
> 
> hist_highP_normChi2.DrawNormalized()
> hist_loose_normChi2.DrawNormalized('same')
> c.SaveAs("track_normChi2.png")
> hist_highP_numPixelHits.DrawNormalized()
> hist_loose_numPixelHits.DrawNormalized('same')
> c.SaveAs("track_nPixelHits.png")
> hist_highP_numValidHits.DrawNormalized()
> hist_loose_numValidHits.DrawNormalized('same')
> c.SaveAs("track_nValHits.png")
> hist_highP_numTkLayers.DrawNormalized()
> hist_loose_numTkLayers.DrawNormalized('same')
> c.SaveAs("track_nTkLayers.png")
> ~~~
> {: .language-python}
> •	track_pt.png:
> <a href="https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png"><img src = "https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png" alt="Efficiency vs pT" width ="350"></a>
> •	track_eta.png:
> <a href="https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png"><img src = "https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png" alt="Efficiency vs pT" width ="350"></a>
> •	track_phi.png:
> <a href="https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png"><img src = "https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png" alt="Efficiency vs pT" width ="350"></a>
> •	track_normChi2.png:
> <a href="https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png"><img src = "https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png" alt="Efficiency vs pT" width ="350"></a>
> •	track_nPixelHits.png:
> <a href="https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png"><img src = "https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png" alt="Efficiency vs pT" width ="350"></a>
> •	track_nTkLayers.png:
> <a href="https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png"><img src = "https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png" alt="Efficiency vs pT" width ="350"></a>
> •	track_nValHits.png:
> <a href="https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png"><img src = "https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/ttbar_phase1_ca_vs_pu_efficiency_pt.png" alt="Efficiency vs pT" width ="350"></a>
{: .solution}
{% include links.md %}

