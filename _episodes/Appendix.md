---
title: "CMS Data Analysis School Tracking and Vertexing Short Exercise - Appendix"
teaching: 0
exercises: 1
questions:
- "How can I use track variables to retrieve the CMS tracking efficiency?"
- "Can I find more about secondary vertex?"
- "Is there a correlation between pile-up and number of clusters?"
objectives:
- "After the in-person session, being familiar with more information about tracks, vertex and CMS tracking detector."
keypoints:
- "At this point it is extremely important to look at the documentation to find out moreabout the Tracking Physics Object Group (TRK POG) tasks."
---
## LXR: a tool to search through CMSSW code
LXR is a very useful tool to look-up methods/classes/configuration files/parameters/example etc. when working on software development in CMS.

* CERN: [https://cmssdt.cern.ch/lxr/](https://cmssdt.cern.ch/lxr/)
* Fermilab: [http://cmslxr.fnal.gov/lxr/](http://cmslxr.fnal.gov/lxr/)

## Further useful code references
*	CMSSW source code on GitHub for `CMSSW_10_6_18`: [https://github.com/cms-sw/cmssw/tree/CMSSW_10_6_18](https://github.com/cms-sw/cmssw/tree/CMSSW_10_6_18)
    *	you can switch to the branch or version (encoded as git tag) using the drop down menu left of the green `New Pull Request` button
*	CMSSW Reference Manual: [https://cmssdt.cern.ch/SDT/doxygen/CMSSW_10_6_18/doc/html/classes.html](https://cmssdt.cern.ch/SDT/doxygen/CMSSW_10_6_18/doc/html/classes.html)

## Tracking efficiency performance via the Tag and Probe technique
The **tag and probe** method is a **data-driven technique** for measuring particle detection efficiencies. It is based on the decays of **known resonances** (e.g. J/ψ, ϒ and Z) to pairs of the particles being studied. In this exercise, these particles are muons, and the Z resonance is nominally used.
The determination of the detector efficiency is a critical ingredient in any physics measurement. It accounts for the particles that were produced in the collision but escaped detection (did not reach the detector elements, were missed by the reconstructions algorithms, etc). It can be in general estimated using simulations, but simulations need to be calibrated with data. The T&P method here described provides a useful and elegant mechanism for extracting efficiencies directly from data!.
### What is “tag” and “probe”?

The resonance, used to calculate the efficiencies, decays to a pair of particles: the tag and the probe.
*	Tag muon = well identified, triggered muon (tight selection criteria).
*	Probe muon = unbiased set of muon candidates (very loose selection criteria), either passing or failing the criteria for which the efficiency is to be measured.
### How do we calculate the efficiency?
The efficiency is given by the fraction of probe muons that pass a given criteria:

* The denominator corresponds to the number of resonance candidates (tag+probe pairs) reconstructed in the dataset. 
* The numerator corresponds to the subset for which the probe passes the criteria.
* The tag+probe invariant mass distribution is used to select only signal, that is, only true Z candidates decaying to dimuons. This is achieved in this exercise by the usage of the **fitting method**.

In this exercise the probe muons are `StandAlone` muons: all tracks of the segments reconstructed in the muon chambers (performed using segments and hits from Drift Tubes - DTs in the barrel region, Cathode strip chambers - CSCs in the endcaps and Resistive Plates Chambers - RPCs for all muon system) are used to generate “seeds” consisting of position and direction vectors and an estimate of the muon transverse momentum. The standalone muon is matched in (ΔR < 0.3, Δη < 0.3) with `generalTracks` in [AOD](https://twiki.cern.ch/twiki/bin/view/CMS/AOD), `lostTracks` in MiniAOD having p<sub>T</sub> larger than 10 [GeV](https://twiki.cern.ch/twiki/bin/view/CMS/GeV), being in this case a **passing probes**.

### The fitting method

It consists on fitting the invariant mass of the tag & probe pairs, in the two categories: passing probes, and all probes. I.e., for the unbiased leg of the decay, one can apply a selection criteria (a set of cuts) and determine whether the object passes those criteria or not.

The procedure is applied after splitting the data in bins of a kinematic variable of the probe object (e.g. the traverse momentum, p<sub>T</sub>); as such, the efficiency will be measured as a function of that quantity for each of the bins.
So, in the picture below, on the left, let’s imagine that the p<sub>T</sub> bin we are selecting is the one marked in red. But, of course, in that bin (like in the rest) you will have true Z decays as well as muon pairs from other processes (maybe QCD, for instance). The true decays would make up our signal, whereas the other events will be considered the background.

The fit, which is made in a different space (the invariant mass space) allows to statistically discriminate between signal and background. To compute the efficiency we simply divide the signal yield from the fits to the passing category by the signal yield from the fit of the inclusive (All) category. This approach is depicted in the middle and right-hand plots of the image below for the Y resonance.

<a href="https://cms-opendata-workshop.github.io/workshop-lesson-tagandprobe/fig/esquema.png"><img src = "https://cms-opendata-workshop.github.io/workshop-lesson-tagandprobe/fig/esquema.png" alt="Tag and Probe method" width ="900"></a>

At the end of this section, then, you will have to make these fits for each bin in the range of interest.
The dataset used in this exercise has been collected by the CMS experiment, in proton-proton collisions at the LHC. It contains `986100 entries` (muon pair candidates) with an associated invariant mass. For each candidate, the transverse `momentum (pt)`, `rapidity(η)` and `azimuthal angle (φ)` are stored, along with a binary flag `probe_isTrkMatch`, which is `1` in case the corresponding probe satisfied the track matching selection criteria and 0 in case it doesn’t.

Copy `CMSDAS_TP` inside `CMSSW_10_6_18/src`:
~~~
cp -r /eos/user/c/cmsdas/2023/short-ex-trk/CMSDAS_TP .
~~~
{: .language-bash}
Exploring the content of the `TP_Z_DATA.root` and `TP_Z_MC.root` files, the `StandAloneEvents` tree has these variables in which we are interested in:
> ## Variables to be checked
> * pair_mass
> * probe_isTrkMatch
> * probe_pt
> * probe_eta
> * probe_phi
{: .checklist}
We’ll start by calculating the efficiency as a function of the probe η. It is useful to have an idea of the distribution of the quantity we want to study. In order to do this, **plot the invariant mass and the probe variables**.

Now that you’re acquainted with the data, open the `Efficiency.C` file. We’ll start by choosing the desired bins for the rapidity. If you’re feeling brave, modify bins for our fit remembering that we need a fair amount of data in each bin (more events mean a better fit!). If not, we’ve left a suggestion in the `Efficiency.C` file. Start with the eta variable.

Now that the bins are set, we have defined the initial parameters for our fit. In this code, we execute a simultaneous fit using a Voigtian curve for the Z peak. For the background we use a Falling Exponential. The function used, doFit(), is implemented in the source file `src/DoFit.cpp` and it was based on the [RooFit](https://root.cern.ch/doc/master/group__Roofit.html) library. You can find generic tutorials for this library [here](https://root.cern.ch/doc/master/group__tutorial__roofit.html). If you’re starting with RooFit you may also find [this one](https://indico.scc.kit.edu/event/31/contributions/1864/attachments/1105/1550/lukas_hehn_kseta-workshop_introduction-to-RooFit.pdf) particularly useful. You won’t need to do anything in `src/DoFit.cpp` but you can check it out if you’re curious.

To get the efficiency plot, we used the `TEfficiency` class from ROOT. You’ll see that in order to create a `TEfficiency object`, one of the constructors requires two `TH1 objects`, i.e., two histograms. One with all the probes and one with the passing probes.

The creation of these `TH1 objects` is taken care of by the `src/make_hist.cpp` code. Note that we load all these functions in the src area directly in header of the `Efficiency.C` code. Now that you understand what the` Efficiency.C` macro does, run your code with in a batch mode `(-b)` and with a quit-when-done switch `(-q)` `root -q -b Efficiency.C`.

When the execution finishes, you should have 2 new files. One on your working directory `Histograms_Data.root` and another one `Efficiency_Run2018.root` located at `Efficiency_Result/eta`. The second contains the efficiency we calculated, while the first file is used to re-do any unusuable fits. If you want, check out the PDF files under the `Fit_Result/` directory, which contain the fitting results as the following one:

<a href="https://raw.githubusercontent.com/CMSTrackingPOG/trackingvertexing/gh-pages/data/probe_eta-0.200000__probe_eta=0.200000_Data-1.png"><img src = "https://raw.githubusercontent.com/CMSTrackingPOG/trackingvertexing/gh-pages/data/probe_eta-0.200000__probe_eta=0.200000_Data-1.png" alt="Fitting procedure applied to the Z di-muon boson invariant mass for both passing and all probes" width ="500"></a>

Now we must re-run the code, but before that, change `IsMc` value to `TRUE`. This will generate an efficiency for the simulated data, so that we can compare it with part of the 2018 run. If so, now uncomment `Efficiency.C` the following line:
~~~
// compare_efficiency(quantity, "Efficiency_Result/eta/Efficiency_Run2018.root", "Efficiency_Result/eta/Efficiency_MC.root");
~~~
{: .language-cpp}

and run the macro again. You should get something like the following result if you inspect the image at `Comparison_Run2018_vs_MC_Efficiency.png`:
<a href="https://raw.githubusercontent.com/CMSTrackingPOG/trackingvertexing/gh-pages/data/Efficiency.png"><img src = "https://raw.githubusercontent.com/CMSTrackingPOG/trackingvertexing/gh-pages/data/Efficiency.png" alt="Tracking Efficiency for CMS 2018 Data" width ="900"></a>

If everything went well and you still have time to go, repeat this process for the two other variables, p<sub>T</sub> and φ! In case you want to change one of the fit results, use the `change_bin.cpp` function commented in `Efficiency.C`. If you would like to explore the results having more statistics, use the samples in `DATA/Z/` directory!

If you would like to play with the actual CMS tracking workflow have a look at the ntuple generator and the efficiency computation frameworks!

## Looking at secondary vertices (continued)

**As an exercise, plot the positions of the vertices in a way that corresponds to the CMS half-view.** That is, make a histogram like the following:
~~~
import math
import ROOT
rho_z_histogram = ROOT.TH2F("rho_z", "rho_z", 100, 0.0, 30.0, 100, 0.0, 10.0)
~~~
{: .language-python}

in which the horizontal axis will represent z, the direction parallel to the beamline, and the vertical axis will represent rho, the distance from the beamline, and fill it with (z, rho) pairs like this `rho_z_histogram.Fill(z, rho)`.

Compute rho and z from the vertex objects:

~~~
events.toBegin()
for event in events:
    event.getByLabel("SecondaryVerticesFromLooseTracks", "Kshort", secondaryVertices)
    for vertex in secondaryVertices.product():
        rho_z_histogram.Fill(abs(vertex.vz()), math.sqrt(vertex.vx()**2 + vertex.vy()**2))

rho_z_histogram.Draw()
~~~
{: .language-python}
> ## Question
> What does the distribution tell you about CMS vertex reconstruction?
{: .discussion}

Half-view of CMS tracker (color indicates average number of hits):
<a href="https://raw.githubusercontent.com/CMSTrackingPOG/trackingvertexing/gh-pages/data/occupancy_map_blueyellow.png"><img src = "https://raw.githubusercontent.com/CMSTrackingPOG/trackingvertexing/gh-pages/data/occupancy_map_blueyellow.png" alt="half-view of CMS tracker" width ="500"></a>

### Correlation between pile-up and number of clusters
~~~
import ROOT
import DataFormats.FWLite as fwlite
events = fwlite.Events("/eos/user/c/cmsdas/2023/short-ex-trk/run321167_ZeroBias_AOD.root")

clusterSummary = fwlite.Handle("ClusterSummary")

h = ROOT.TH2F("h", "h", 100, 0, 20000, 100, 0, 100000)

events.toBegin()
for event in events:
    event.getByLabel("clusterSummaryProducer", clusterSummary)
    cs = clusterSummary.product()
    try:
        h.Fill(cs.getNClus(cs.PIXEL),
               cs.getNClus(cs.PIXEL) + cs.getNClus(cs.STRIP))
    except TypeError:
        pass

c = ROOT.TCanvas("c", "c", 800, 800)
h.Draw()
h.Fit("pol1")
c.SaveAs("pileup_nclusters.png")
~~~
{: .language-python}

### Harder questions: dxy biases

The dxy parameter is not simply a distance, it is a signed distance. A helical trajectory traces a circle in the plane transverse to the beamline, and the sign of dxy depends on whether the reference point is included inside of that circle our outside of it. You can change the reference point with which dxy is computed by passing a point or a beamspot as an argument:
~~~
print track.dxy()
~~~
{: .language-python}
~~~
-0.00082122773835
~~~
{: .output}
~~~
print track.dxy(ROOT.math.XYZPoint(0, 0, 0))
~~~
{: .language-python}
~~~
-0.00082122773835
~~~
{: .output}
~~~
print track.dxy(beamspot.product())
~~~
{: .language-python}
~~~
-0.00636893586021
~~~
{: .output}

Passing no parameter is equivalent to passing `(0, 0, 0)`.
Consider the following script:
~~~
import DataFormats.FWLite as fwlite
import ROOT

events = fwlite.Events("/eos/user/c/cmsdas/2023/short-ex-trk/run321167_ZeroBias_AOD.root")
tracks = fwlite.Handle("std::vector<reco::Track>")
beamspot = fwlite.Handle("reco::BeamSpot")

dxy_vs_phi_000 = ROOT.TProfile("dxy_vs_phi_000", "dxy_vs_phi_000", 100, -3.14, 3.14) 
dxy_vs_phi_beamspot = ROOT.TProfile("dxy_vs_phi_beamspot", "dxy_vs_phi_beamspot", 100, -3.14, 3.14) 

events.toBegin()
for event in events:
    event.getByLabel("generalTracks", tracks)
    event.getByLabel("offlineBeamSpot", beamspot)
    for track in tracks.product():
        dxy_vs_phi_000.Fill(track.phi(), track.dxy(ROOT.math.XYZPoint(0, 0, 0)))
        dxy_vs_phi_beamspot.Fill(track.phi(), track.dxy(beamspot.product()))

dxy_vs_phi_000.SetAxisRange(-0.2, 0.2, "Y")
dxy_vs_phi_beamspot.SetAxisRange(-0.2, 0.2, "Y")

c = ROOT.TCanvas("c", "c", 800, 800)
dxy_vs_phi_000.Draw()
dxy_vs_phi_beamspot.SetMarkerColor(ROOT.kRed)
dxy_vs_phi_beamspot.SetLineColor(ROOT.kRed)
dxy_vs_phi_beamspot.Draw("same")
c.SaveAs("beamspot.png")
~~~
{: .language-python}

It produces two plots, one of which shows a clear bias from the correct distribution (on average, dxy should be zero). This bias is not an problem with the detector, only a problem with interpreting it. Can you explain the origin of the bias, including its shape with respect to phi?

Below, there is a figure that might help.
It is also useful to draw sketches for yourself.

<a href="https://raw.githubusercontent.com/CMSTrackingPOG/trackingvertexing/gh-pages/data/dxy.png"><img src = "https://raw.githubusercontent.com/CMSTrackingPOG/trackingvertexing/gh-pages/data/dxy.png" alt="Sketch of dxy" width ="200"></a>


## Bonus Exercise: µµTrkTrk reconstruction in MiniAOD

Please have a look to the instruction at the repository at [https://github.com/CMSTrackingPOG/2mu2trk_exercise/blob/main/README.md](https://github.com/CMSTrackingPOG/2mu2trk_exercise/blob/main/README.md).

Please notice that it requires a newer CMSSW version! 

## The setup
This package is meant to be run using Ultra Legacy MINIAODv2

* Setup in `13_1_0` 

~~~
scram p -n cmssw CMSSW_13_1_0
cd cmssw/src/
cmsenv
git clone git@github.com:CMSTrackingPOG/2mu2trk_exercise.git -b jpsiphi MuMuTrkTrk/MuMuTrkTrk/
scram b -j 8
~~~
{: .language-bash}

To run the example on `/RelValBsToJpsiPhi_mumuKK_14TeV/CMSSW_13_0_0_pre3-130X_mcRun3_2022_realistic_v2-v1/MINIAODSIM`

If you haven't, set up your certificate to access the data:
~~~
voms-proxy-init -rfc -voms cms -valid 192:00
~~~
{: .language-bash}

Then simply run the config
~~~
cmsRun MuMuTrkTrk/MuMuTrkTrk/test/run-jpsikk-miniaodsim.py
~~~
{: .language-bash}


## The Decay

The decay we want to reconstruct a $B^0_s$ meson candidate decaying as:

$$B_s^0 \to J/\psi(\to \mu\mu)\phi(\to KK)  $$

a bottom strange meson ($s\bar{b}$). You can find further information in the [PDG](https://pdg.lbl.gov/2023/web/viewer.html?file=../tables/rpp2023-tab-mesons-bottom-strange.pdf).

<a href="https://raw.githubusercontent.com/CMSTrackingPOG/2mu2trk_exercise/main/images/bs_meson.png"><img src = "https://raw.githubusercontent.com/CMSTrackingPOG/2mu2trk_exercise/main/images/bs_meson.png" alt="BS Meson." width ="500"></a>

We can sketch the decay topology as follows:

<a href="https://raw.githubusercontent.com/CMSTrackingPOG/2mu2trk_exercise/main/images/bs_jps_phi.png"><img src = "https://raw.githubusercontent.com/CMSTrackingPOG/2mu2trk_exercise/main/images/bs_jps_phi.png" alt="bs_jspi_phi" width ="500"></a>

the $B_s^0$ candidate, before decaying, flies (give the non negligible lifetime) w.r.t the collision area (beam spot) and it decays in:
- $J/psi$ meson, a $c\bar{c}$ bound state with a mass of $\sim 3096 MeV$(see the [PDG](https://pdg.lbl.gov/2023/web/viewer.html?file=../tables/rpp2023-tab-mesons-c-cbar.pdf) for further details) 
- $\phi$ meson ($s\bar{s}$) a mass of $\sim 1020 MeV$(see the [PDG](https://pdg.lbl.gov/2023/tables/contents_tables_mesons.html ) for further details).

The $J/psi$ and the $\phi$ then furhterly decay into a pair of opposite sign muons and a pair of opposite sign kaons, almost promptly w.r.t. the $B^0_s$ meson.

## The Analyzer

### The Config

We select the global tag suitable for the dataset we have
~~~
process.GlobalTag = GlobalTag(process.GlobalTag, '130X_mcRun3_2022_realistic_v2', '')
~~~
{: .language-python}

this is something you can usually get from the dataset name itself.

Next step let's configure the inputs and the outputs with the `PoolSource` module and the `TFileService` (used to output a simple `ROOT` file).

~~~
input_filename = ["/store/relval/CMSSW_13_0_0_pre3/RelValBsToJpsiPhi_mumuKK_14TeV/MINIAODSIM/130X_mcRun3_2022_realistic_v2-v1/00000/d26ae0bf-3a06-4098-9c24-c29452079aa0.root"]
ouput_filename = 'mumukk.root'
process.source = cms.Source("PoolSource",fileNames = cms.untracked.vstring(input_filename))
process.TFileService = cms.Service("TFileService",fileName = cms.string(ouput_filename))
~~~
{: .language-python}

Then we may select the events that has fired a specific trigger:

~~~
triggers = [
'HLT_DoubleMu4_JpsiTrkTrk_Displaced', ##Run3 trigger!
]
~~~
{: .language-python}

this is a trigger dedicated to $J/\psi + 2 Tracks$ displaced topologies (exactly what we need!). The $4$ there is the $p_T$ threshold on the $2\mu$ system. So we add to the process a module to filter the events:

~~~
hltpathsV = cms.vstring([h + '_v*' for h in triggers ])
process.triggerSelection = cms.EDFilter("TriggerResultsFilter",
                                        triggerConditions = hltpathsV,
                                        hltResults = cms.InputTag( "TriggerResults", "", "HLT" ),
                                        l1tResults = cms.InputTag( "" ),
                                        throw = cms.bool(False)
                                        )

~~~
{: .language-python}

In `MINIAOD` the muons are stored under the `slimmedMuons` collection of `pat::Muons`. Instead of using all of them we may run a first selection based on a combination of cuts with the `PATMuonSelector` module: 

~~~
process.oniaSelectedMuons = cms.EDFilter('PATMuonSelector',
   src = cms.InputTag('slimmedMuons'),
   cut = cms.string('muonID(\"TMOneStationTight\")'
                    ' && abs(innerTrack.dxy) < 0.3'
                    ' && abs(innerTrack.dz)  < 20.'
                    ' && innerTrack.hitPattern.trackerLayersWithMeasurement > 5'
                    ' && innerTrack.hitPattern.pixelLayersWithMeasurement > 0'
                    ' && innerTrack.quality(\"highPurity\")'
                    ' && (pt > 2.)'
   ),
   filter = cms.bool(True)
)
~~~
{: .language-python}

In this case, e.g., we are selecting muons:
- `innerTrack.hitPattern.pixelLayersWithMeasurement>0`, whose track has at least one pixel on at least one layer;
- `innerTrack.hitPattern.pixelLayersWithMeasurement>0`, whose track has at least one hit on at least five layers;
- `innerTrack.quality(\"highPurity\")`, an high purity track
- `pt > 2.`, etc...


Now it's the moment to build the µµ candidate:

~~~
process.load("MuMuTrkTrk.MuMuTrkTrk.onia2MuMuPAT_cfi")
~~~
{: .language-python}


~~~
process.onia2MuMuPAT.muons=cms.InputTag('oniaSelectedMuons')
~~~
{: .language-python}

~~~
process.onia2MuMuPAT.dimuonSelection=cms.string("2.5 < mass && mass < 3.5")
~~~
{: .language-python}

### The Notebook

The `bs_decay.ipynb` notebook under `MuMuTrkTrk/MuMuTrkTrk/test/`

It's easier if you follow the instructions directly there. If you want you may run this notebook in [SWAN](https://swan.web.cern.ch/swan/) from which you will be able to acces any area you have acces on `eos` (e.g. your `lxplus` home).


 <a href="https://cern.ch/swanserver/cgi-bin/go?projurl=https://raw.githubusercontent.com/CMSTrackingPOG/2mu2trk_exercise/main/test/bs_decay.ipynb" target="_blank">
                            <img src="https://swanserver.web.cern.ch/swanserver/images/badge_swan_white_150.png" alt="Open in SWAN" style="height:1.5em">
                        </a>

## To you!

As an excercise you may try to modify the analyzer chain and the python config in order to reconstruct another decay with a similar daughters:

$$\psi(2S) \to J/\psi(\to \mu\mu)\pi\pi  $$

<a href="https://raw.githubusercontent.com/CMSTrackingPOG/2mu2trk_exercise/main/images/psi2s.png"><img src = "https://raw.githubusercontent.com/CMSTrackingPOG/2mu2trk_exercise/main/images/psi2s.png" alt="psi2s" width ="500"></a>

There are some differences:

1. in this case the decay is __prompt__, meaning we expect all the candidates to come from a PV;
2. there is no constraint on the $\pi\pi$ system mass since they are non-resonant;
3. the $\psi(2S)$ has a lower mass w.r.t. the $B^0_s$

You can try to use the dataset

`/RelValPsi2SToJPsiPiPi_14TeV/CMSSW_13_0_0_pre3-130X_mcRun3_2022_realistic_v2-v1/MINIAODSIM`

that has the same GT.

(Suggestion: also the `HLT` you select will need to be changed, if you don't know which to use, simply turn off the selection.)

Another possibility is to check the PU datasets for $B^0_s$ such as:

`/RelValBsToJpsiPhi_mumuKK_14TeV/CMSSW_13_1_0_pre4-PU_131X_mcRun3_2022_realistic_v3_2023_BPH-v1/MINIAODSIM`

and see what happens. Beware! The GT is different!

## Documentation
*	[TRK POG twiki](https://twiki.cern.ch/twiki/bin/view/CMSPublic/PhysicsResultsTRK)
*	[CMS Tracking POG Performance in Run-2 Legacy data](https://twiki.cern.ch/twiki/bin/view/CMSPublic/TrackingPOGResultsRun2Legacy)


{% include links.md %}

