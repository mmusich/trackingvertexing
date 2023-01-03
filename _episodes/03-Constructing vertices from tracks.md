---
title: "Constructing vertices from tracks"
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
In this exercise we will encounter three main objects: primary vertices, secondary vertices, and the beamspot.
Let's start with secondary vertices. Particles produced by a single decay or collision radiate from the point of their origin. Their tracks should nearly cross at this point (within measurement uncertainties): if two or more tracks cross at some point, it is very likely that they descend from the same decay or collision.
This test is more significant than it may appear by looking at event pictures. With so many tracks, it looks like they cross accidentally, but the two-dimensional projection of the event picture is misleading. One-dimensional paths through three-dimensional space do not intersect easily, especially when the trajectories of those paths are measured with microns to hundreds-of-microns precision.
Starting from the detected tracks, we work backward and reconstruct the original vertices by checking each pair of tracks for overlaps. This is performed in the standard reconstruction sequence and delivered to the analyst as lists of KS → π+π− and Λ → pπ candidates, but we will repeat the procedure with different parameters. The algorithm will run three times, the first accepting all (`loose`) tracks, the second accepting only `tight` tracks, and the third accepting only `highPurity` tracks.
These are called secondary vertices because the proton-proton collision produced the first (`primary`) vertex, then the neutral KS flew several centimeters away from the rest of the collision products and decayed into π+π− at a second (`secondary`) position in space.
## Running the vertex reconstruction
Create a file named `construct_secondary_vertices_cfg.py` in `TrackingShortExercize/` and fill it with the following:
~~~
import FWCore.ParameterSet.Config as cms

process = cms.Process("KSHORTS")

# Use the tracks_and_vertices.root file as input.
process.source = cms.Source("PoolSource",
    fileNames = cms.untracked.vstring("file://run321167_Charmonium_AOD.root"))
process.maxEvents = cms.untracked.PSet(input = cms.untracked.int32(500))

# Suppress messages that are less important than ERRORs.
process.MessageLogger = cms.Service("MessageLogger",
    destinations = cms.untracked.vstring("cout"),
    cout = cms.untracked.PSet(threshold = cms.untracked.string("ERROR")))

# Load part of the CMSSW reconstruction sequence to make vertexing possible.
# We'll need the CMS geometry and magnetic field to follow the true, non-helical
# shapes of tracks through the detector.
process.load("Configuration/StandardSequences/FrontierConditions_GlobalTag_cff")
from Configuration.AlCa.GlobalTag import GlobalTag
process.GlobalTag =  GlobalTag(process.GlobalTag, "auto:run2_data")
process.load("Configuration.Geometry.GeometryRecoDB_cff")
process.load("Configuration.StandardSequences.MagneticField_cff")
process.load("TrackingTools.TransientTrack.TransientTrackBuilder_cfi")

# Copy most of the vertex producer's parameters, but accept tracks with
# progressively more strict quality.
process.load("RecoVertex.V0Producer.generalV0Candidates_cfi")

# loose
process.SecondaryVerticesFromLooseTracks = process.generalV0Candidates.clone(
    trackRecoAlgorithm = cms.InputTag("generalTracks"),
    doKshorts = cms.bool(True),
    doLambdas = cms.bool(True),
    trackQualities = cms.string("loose"),
    innerHitPosCut = cms.double(-1.),
    cosThetaXYCut = cms.double(-1.), 
    )

# tight
process.SecondaryVerticesFromTightTracks = process.SecondaryVerticesFromLooseTracks.clone(
    trackQualities = cms.string("tight"),
    )

# highPurity
process.SecondaryVerticesFromHighPurityTracks = process.SecondaryVerticesFromLooseTracks.clone(
    trackQualities = cms.string("highPurity"),
    )

# Run all three versions of the algorithm.
process.path = cms.Path(process.SecondaryVerticesFromLooseTracks *
                        process.SecondaryVerticesFromTightTracks *
                        process.SecondaryVerticesFromHighPurityTracks)

# Writer to a new file called output.root.  Save only the new K-shorts and the
# primary vertices (for later exercises).
process.output = cms.OutputModule(
    "PoolOutputModule",
    SelectEvents = cms.untracked.PSet(SelectEvents = cms.vstring("path")),
    outputCommands = cms.untracked.vstring(
        "drop *",
        "keep *_*_*_KSHORTS",
        "keep *_offlineBeamSpot_*_*",
        "keep *_offlinePrimaryVertices_*_*",
        "keep *_offlinePrimaryVerticesWithBS_*_*",
        ),
    fileName = cms.untracked.string("output.root")
    )
process.endpath = cms.EndPath(process.output)
~~~
{: .language-python}
The name of the secondary-vertex producer (`process.SecondaryVerticesFromLooseTracks`) given in this configuration will be the name of the produced collection with process. stripped off (see the usage below).
Run the configuration file with:
~~~
cmsRun construct_secondary_vertices_cfg.py
~~~
{: .language-bash}

which can take a couple of minutes to complete. Notice it will create the `output.root` file which will be used later.
You can check the content of the file by running the simple script as follows
~~~
edmDumpEventContent output.root
~~~
{: .language-bash}
## Looking at secondary vertices
When it's done, you can open `output.root` and read the vertex positions, just as you did for the track momenta with the following lines (let's name this code `sec_vertices.py` and put it in `TrackingShortExercize/`):
~~~
import DataFormats.FWLite as fwlite

events = fwlite.Events("file:output.root")
secondaryVertices = fwlite.Handle("std::vector<reco::VertexCompositeCandidate>")

events.toBegin()
for i, event in enumerate(events):
    print "Event:", i
    event.getByLabel("SecondaryVerticesFromLooseTracks", "Kshort", secondaryVertices)
    for j, vertex in enumerate(secondaryVertices.product()):
        print "    Vertex:", j, vertex.vx(), vertex.vy(), vertex.vz()
    if i > 10: break
~~~
{: .language-python}

In the code above, you specify the C++ type of the collection (`std::vector`). For each event you obtain the collection of secondary vertices originating from KS decays, by providing the name of the collection producer (`SecondaryVerticesFromLooseTracks`) and the label for the KS collection (`Kshort`, defined [here](https://github.com/cms-sw/cmssw/blob/CMSSW_8_0_10_patch2/RecoVertex/V0Producer/src/V0Producer.cc#L55)).

Each of these vertices contains two tracks by construction. One of the vertex member functions (do `dir(vertex)` in the python shell after executing the above code to see them all) returns the invariant mass of the pair of tracks. This calculation is a little more complex than the invariant masses we calculated by hand because it is necessary to evaluate the px and py components of momentum at the vertex rather than at the beamspot. If these two particles really did originate in a decay several centimeters from the beamspot, then the track parameters evaluated at the beamspot are not meaningful.

**Plot the invariant mass of all vertices**. Modify the `sec_vertices.py` to draw the invariant mass distribution, i.e. fill a histogram with `vertex.mass()` for each secondary vertex.
> ## Answer (don't look until you've tried it!)
> ~~~
> import DataFormats.FWLite as fwlite
> import ROOT
> 
> events = fwlite.Events("file:output.root")
> secondaryVertices = fwlite.Handle("std::vector<reco::VertexCompositeCandidate>")
> 
> mass_histogram = ROOT.TH1F("mass", "mass", 100, 0.4, 0.6)
> 
> events.toBegin()
> for event in events:
>     event.getByLabel("SecondaryVerticesFromLooseTracks", "Kshort", secondaryVertices)
>     for vertex in secondaryVertices.product():
>         mass_histogram.Fill(vertex.mass())
> 
> c = ROOT.TCanvas ("c" , "c", 800, 800)
> mass_histogram.Draw()
> c.SaveAs("kshort_mass.png")
> ~~~
> {: .language-python}
{: .solution}
You should see a very prominent KS → π+π− peak, but also a pedestal. What is the pedestal? Why does it cut off at 0.43 and 0.57 [GeV](https://twiki.cern.ch/twiki/bin/view/CMS/GeV)?
> ## More
> You can answer the question concerning the cut-off with the information [here](https://github.com/cms-sw/cmssw/blob/CMSSW_8_0_10_patch2/RecoVertex/V0Producer/python/generalV0Candidates_cfi.py#L61-L63).
> **Optional task.** Prepare a similar plot for the Lambda vertices. (Hints: were the Lambda vertices created when you ran the [SecondaryVertices](https://twiki.cern.ch/twiki/bin/edit/CMS/SecondaryVertices) producer? the Lambda mass is [1.115 GeV](http://pdglive.lbl.gov/Particle.action?init=0&node=S018&home=BXXX020) Are you expecting to reconstruct the mass at the same value in the whole detector ? (plot the mass resolution as function of the eta of the reconstructed Lambda)
{: .solution}
**Plot the flight distance of all vertices.** Modify the `sec_vertices.py` to draw the flight distance in the transverse plane distribution between each secondary vertex and the primary vertex. For this, you can look ahead at the next part of the exercise to see how to access the primary vertices collection. Once you have it, you can access the first primary vertex in the collection with `pv = primaryVertices.product()[0]`. 

You can now use the x and y coordinates of the secondary vertices and the primary vertex to calculate the distance. Note that for the PV, you access these values with the `x()` and `y()` member functions, while for the secondary vertices, these are called `vx()` and `vy()`.

> ## Answer
> ~~~
> import DataFormats.FWLite as fwlite
> import ROOT
> 
> events = fwlite.Events("file:output.root")
> secondaryVertices = fwlite.Handle("std::vector<reco::VertexCompositeCandidate>")
> primaryVertices = fwlite.Handle("std::vector<reco::Vertex>")
> 
> lxy_histogram = ROOT.TH1F("lxy", "lxy", 500, 0, 70)
> 
> events.toBegin()
> for event in events:
>     event.getByLabel("offlinePrimaryVertices", primaryVertices)
>     pv = primaryVertices.product()[0]
>     event.getByLabel("SecondaryVerticesFromLooseTracks", "Lambda", secondaryVertices)
>     for vertex in secondaryVertices.product():
>         lxy = ((vertex.vx()-pv.x())**2 + (vertex.vy() - pv.y())**2)**0.5
>         lxy_histogram.Fill(lxy)
> c = ROOT.TCanvas ("c" , "c", 800, 800)
> lxy_histogram.Draw()
> c.SaveAs("lambda_lxy.png")
> ~~~
> {: .language-python}
{: .solution}
You should rerun the `construct_secondary_vertices.py` to process all events in the file to get more statistics. For this, set the maxEvents parameter to `-1`. Running on more events, one can appreciate some structures in the flight distance distribution, can you explain them ? Note that these are especially noticeable in the distribution of Lambdas.
<a href="https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/v0_Lxy.png"><img src = "https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/v0_Lxy.png" alt="Lxy distribution" width ="200"></a>

{% include links.md %}

