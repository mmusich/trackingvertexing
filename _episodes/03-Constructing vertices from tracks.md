---
title: "CMS Data Analysis School Tracking and Vertexing Short Exercise - Constructing vertices from tracks"
teaching: 20
exercises: 20
questions:
- "What is a (primary, secondary) vertex object? What about beamspot?"
- "Which information are available in different CMS data formats?"
- "How can I run the vertex reconstruction?"
- "Can we get better physics results using primary verteces?"
objectives:
- "Being familiar with the vertex concept."
- "Plot distributions about the main vertex variables."
- "Open the stage for Appendix exercises." 
keypoints:
- "Two or more tracks can be used to reconstruct their common point of origin, the vertex"
- "Requiring two tracks to originate from the same vertex is a powerful tool to identify particles that decayed in the detector"
- "The distribution of the proton proton interaction, the so-called interaction region, in the center of the CMS detector is an ellipse."
---
In this exercise, we will encounter three main objects: **primary vertices**, **secondary vertices**, and the **beamspot**.
Let's start with **secondary vertices**. Particles produced by a single decay or collision radiate from the point of their origin. Their tracks should nearly cross at this point (within measurement uncertainties): if two or more tracks cross at some point, it is very likely that they descend from the same decay or collision.

This test is more significant than it may appear by looking at event pictures. With so many tracks, it looks like they cross accidentally, but the two-dimensional projection of the event picture is misleading. One-dimensional paths through three-dimensional space do not intersect easily, especially when the trajectories of those paths are measured with microns to hundreds-of-microns precision.

Starting from the detected tracks, we work backward and reconstruct the original vertices by checking each pair of tracks for overlaps. This is performed in the standard reconstruction sequence and delivered to the analyst as lists of K<sub>S</sub> → π<sup>+</sup>π<sup>-</sup> and Λ → pπ<sup>0</sup> candidates, but we will repeat the procedure with different parameters. The algorithm will run three times, the first accepting all (`loose`) tracks, the second accepting only `tight` tracks, and the third accepting only `highPurity` tracks.

These are called **secondary vertices** because the proton-proton collision produced the first (`primary`) vertex, then the neutral K<sub>S</sub> flew several centimeters away from the rest of the collision products and decayed into π<sup>+</sup>π<sup>-</sup> at a second (`secondary`) position in space.
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

In the code above, you specify the C++ type of the collection (`std::vector`). For each event you obtain the collection of secondary vertices originating from K<sub>S</sub> decays, by providing the name of the collection producer (`SecondaryVerticesFromLooseTracks`) and the label for the K<sub>S</sub> collection (`Kshort`, defined [here](https://github.com/cms-sw/cmssw/blob/CMSSW_8_0_10_patch2/RecoVertex/V0Producer/src/V0Producer.cc#L55)).

Each of these vertices contains two tracks by construction. One of the vertex member functions (do `dir(vertex)` in the python shell after executing the above code to see them all) returns the invariant mass of the pair of tracks. This calculation is a little more complex than the invariant masses we calculated by hand because it is necessary to evaluate the px and py **components of momentum at the vertex** rather than at **the beamspot**. If these two particles really did originate in a decay several centimeters from the beamspot, then the **track parameters evaluated at the beamspot are not meaningful**.

> ## Question 1
> **Plot the invariant mass of all vertices**. Modify the `sec_vertices.py` to draw the invariant mass distribution, i.e. fill a histogram with `vertex.mass()` for each secondary vertex.
{: .challenge}
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
> ## Question 2
> You should see a very prominent K<sub>S</sub> → π<sup>+</sup>π<sup>-</sup> peak, but also a pedestal. **What is the pedestal?** Why does it cut off at 0.43 and 0.57 [GeV](https://twiki.cern.ch/twiki/bin/view/CMS/GeV)?
{: .discussion}
> ## More
> You can answer the question concerning the cut-off with the information [here](https://github.com/cms-sw/cmssw/blob/CMSSW_8_0_10_patch2/RecoVertex/V0Producer/python/generalV0Candidates_cfi.py#L61-L63).
{: .solution}
> ## Question 3
> Prepare a similar **plot for the Lambda vertices**. (Hints: were the Lambda vertices created when you ran the [SecondaryVertices](https://twiki.cern.ch/twiki/bin/edit/CMS/SecondaryVertices) producer? the Lambda mass is [1.115 GeV](http://pdglive.lbl.gov/Particle.action?init=0&node=S018&home=BXXX020). Are you expecting to reconstruct the mass at the same value in the whole detector ? (optionally plot the mass resolution as function of the eta of the reconstructed Lambda).
{: .discussion}

> ## Question 4
> **Plot the flight distance of all vertices.** Modify the `sec_vertices.py` to draw the flight distance in the transverse plane distribution between each secondary vertex and the primary vertex (PV). For this, you can look ahead at the next part of the exercise to see how to access the primary vertices collection. Once you have it, you can access the first primary vertex in the collection with `pv = primaryVertices.product()[0]`. 
> You can now use the x and y coordinates of the secondary vertices and the primary vertex to calculate the distance. Note that for the PV, you access these values with the `x()` and `y()` member functions, while for the secondary vertices, these are called `vx()` and `vy()`.
{: .challenge}
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
> ## Question 5
You should rerun the `construct_secondary_vertices.py` to process all events in the file to get more statistics. For this, set the `maxEvents` parameter to `-1`. Running on more events, one can appreciate some structures in the flight distance distribution, can you explain them ? Note that these are especially noticeable in the distribution of Lambdas.
<a href="https://raw.githubusercontent.com/bdanzi/trackingvertexing/gh-pages/data/v0_Lxy.png"><img src = "https://raw.githubusercontent.com/bdanzi/trackingvertexing/gh-pages/data/v0_Lxy.png" alt="Lxy distribution" width ="500"></a>
{: .discussion}
## Secondary vertices in MiniAOD
The [SecondaryVertex](https://twiki.cern.ch/twiki/bin/edit/CMS/SecondaryVertex) collection is already available in the MiniAOD files. You can retrieve the collection type and format by doing:
 ~~~
 xrdcp root://cmseos.fnal.gov//store/user/cmsdas/2023/short_exercises/trackingvertexing/run321167_Charmonium_MINIAOD.root .
 edmDumpEventContent run321167_Charmonium_MINIAOD.root | grep Vertices
 ~~~
 {: .language-bash}

> ## Question 6
> Modify `sec_vertices.py` to read the lambda vertices from the MiniAOD file. You will have to modify the collection type and label accordingly to the MiniAOD content.
{: .challenge}
> ## Answer
> ~~~
> import DataFormats.FWLite as fwlite
> import ROOT
> 
> events = fwlite.Events("run321167_Charmonium_MINIAOD.root")
> secondaryVertices = fwlite.Handle("std::vector<reco::VertexCompositePtrCandidate>")
> 
> mass_histogram = ROOT.TH1F("mass_histogram", "mass_histogram", 100, 1., 1.2)
> 
> events.toBegin()
> for i, event in enumerate(events):
>     event.getByLabel("slimmedLambdaVertices", "", secondaryVertices)
>     for j, vertex in enumerate(secondaryVertices.product()):
>         mass_histogram.Fill(vertex.mass())
> 
> c = ROOT.TCanvas()
> mass_histogram.Draw()
> c.SaveAs("mass_lambda_miniaod.png")
> ~~~
> {: .language-python}
{: .solution}

## Basic distributions of primary vertices
The primary vertex reconstruction consists of three steps:
> ## Steps of primary vertex reconstruction
> *	selection of tracks
> *	clustering of the tracks that appear to originate from the same interaction vertex
> *	fitting for the position of each vertex using its associated tracks
{: .checklist}
All the primary vertices reconstructed in an event are saved in the `reco::Vertex` collection labeled `offlinePrimaryVertices`. Create the file `vertex.py` in `TrackingShortExercize/` which will load the original `run321167_ZeroBias_AOD.root` file and make a quarter-view plot of the vertex distribution (run it using `python vertex.py`):
~~~
import DataFormats.FWLite as fwlite
import math
import ROOT

events = fwlite.Events("file:run321167_ZeroBias_AOD.root")
primaryVertices = fwlite.Handle("std::vector<reco::Vertex>")

rho_z_histogram = ROOT.TH2F("rho_z", "rho_z", 100, 0.0, 30.0, 100, 0.0, 10.0)

events.toBegin()
for event in events:
    event.getByLabel("offlinePrimaryVertices", primaryVertices)
    for vertex in primaryVertices.product():
        rho_z_histogram.Fill(abs(vertex.z()),
                             math.sqrt(vertex.x()**2 + vertex.y()**2))

c = ROOT.TCanvas("c", "c", 800, 800)
rho_z_histogram.Draw()
c.SaveAs("rho_z.png")
~~~
{: .language-python}

You should see a broad distribution in z, close to rho = 0 (much closer than for the secondary vertices, see [appendix](https://bdanzi.github.io/trackingvertexing/Appendix/index.html)).In fact, the distribution is about 0.1 cm wide and 4 cm long. The broad distribution in z is helpful: if 20 primary vertices are uniformly distributed along a 4 cm pencil-like region, we only need 2 mm vertex resolution to distinguish neighboring vertices. Fortunately, the CMS vertex resolution is better than this (better than 20μm and 25μm in x and z, respectively [TRK-11-001](http://inspirehep.net/record/1298029), [performances with the Phase1 pixel detector](https://twiki.cern.ch/twiki/bin/edit/CMS/CMSPublic.TrackingPOGPerformance2017MC#Vertex_Resolutions)), so they can be distinguished with high significance.

To see this, you should modify properly the previous code to make a plot of the distance between primary vertices:
~~~
deltaz_histogram = ROOT.TH1F("deltaz", "deltaz", 1000, -20.0, 20.0)

events.toBegin()
for event in events:
    event.getByLabel("offlinePrimaryVertices", primaryVertices)
    pv = primaryVertices.product()
    for i in xrange(pv.size() - 1):
        for j in xrange(i + 1, pv.size()):
            deltaz_histogram.Fill(pv[i].z() - pv[j].z())

c = ROOT.TCanvas ("c", "c", 800, 800)
deltaz_histogram.Draw()
c.SaveAs("deltaz.png")
~~~
{: .language-python}

The broad distribution is due to the spread in primary vertex positions. Zoom in on the narrow dip near deltaz = 0. This region is empty because if two real vertices are too close to each other, they will be misreconstructed as a single vertex. Thus, there are no reconstructed vertices with such small separations.

> ## Question 7
> **Write a short script to print out the number of primary vertices in each event.** When people talk about the **pile-up**, it is this number they are referring to. If you want, you can even plot it; the distribution should roughly fit a Poisson distribution.
{: .challenge}
> ## Answer
> ~~~
> events.toBegin()
> for i, event in enumerate(events):
>     event.getByLabel("offlinePrimaryVertices", primaryVertices)
>     print "Pile-up:", primaryVertices.product().size()
>     if i > 100: break
> ~~~
> {: .language-python}
{: .solution}
**Print out the number of tracks in a single vertex object.** (Use the trick for vertex in `primaryVertices.product()`: break to obtain a primary vertex object.)
> ## Solution
> ~~~
> print vertex.nTracks()
> print vertex.tracksSize()
> ~~~
> {: .language-python}
> Why might they be different? ( See [tracksSize](https://cmssdt.cern.ch/dxr/CMSSW/source/DataFormats/VertexReco/interface/Vertex.h#94) vs [nTracks](https://cmssdt.cern.ch/dxr/CMSSW/source/DataFormats/VertexReco/interface/Vertex.h#170)).
> `Note:` in C++, you could loop over the tracks associated with this vertex, but this functionality doesn't work in Python.
> In the standard analysis workflow there are many quality requirements to be applied to the events and to the reconstructed quantities in an event. One of these requests is based on the characteristics of the reconstructed primary vertices, and it is defined by the `CMSSW EDFilter` `goodOfflinePrimaryVertices_cfi.py` .
> Are these selections surprising you ?
{: .solution}
> ## Question 8
**Write a short script to plot the distribution of the number of tracks vs the number of vertices.** What do you expect?
{: .challenge}
> ## Answer
> The following snippet tells you what to do, but you need to make sure that all needed variables are defined.
> ~~~
> histogram = ROOT.TH2F("ntracks_vs_nvertex","ntracks_vs_nvertex",
>                       30, 0.0, 29.0, 100, 0.0,2000.0)
> 
> events.toBegin()
> for event in events:
>     event.getByLabel("offlinePrimaryVertices", primaryVertices)
>     event.getByLabel("generalTracks", tracks)
>     histogram.Fill(primaryVertices.product().size(), tracks.product().size())
> 
> histogram.Draw()
> c.SaveAs("ntracks_vs_nvertex.png")
> ~~~
> {: .language-python}
{: .solution}
It's also useful to distinguish between the `primary vertices` and the `beamspot`. A read-out event can have many primary vertices, each of which usually corresponds to a point in space where two protons collide. The beamspot is an estimate of where protons are expected to collide, derived from the distribution of primary vertices. Not only does an event have only one beamspot, but the beamspot is constant for a lumi-section (time interval of 23.31 consecutive seconds). This script prints the average x-position of the primary vertices and plots their x-y distribution (add the missing pieces, name it `beamspot.py` and put it in `TrackingShortExercize`):
~~~
import DataFormats.FWLite as fwlite
import math
import ROOT

def isGoodPV(vertex):
    if ( vertex.isFake()        or 
         vertex.ndof < 4.0      or 
         abs(vertex.z()) > 24.0 or 
         abs(vertex.position().Rho()) > 2):
           return False
    return True

events          = fwlite.Events("run321167_ZeroBias_AOD.root")
primaryVertices = fwlite.Handle("std::vector<reco::Vertex>")
beamspot        = fwlite.Handle("reco::BeamSpot")

vtx_xy = ROOT.TH2F('vtx_xy','; x [cm]; y [cm]', 100,-0.1, 0.3, 100, -0.25, 0.15)

sumx = 0.0
N    = 0
last_beamspot = None

events.toBegin()
for event in events:
    event.getByLabel("offlinePrimaryVertices", primaryVertices)
    event.getByLabel("offlineBeamSpot", beamspot)

    if last_beamspot == None or last_beamspot != beamspot.product().x0():
        print "New beamspot IOV (interval of validity)..."
        last_beamspot       = beamspot.product().x0()
        sumx = 0.0
        N = 0

    for vertex in primaryVertices.product():
        if not isGoodPV(vertex):  continue
        N += 1
        sumx += vertex.x()
        vtx_xy.Fill(vertex.x(), vertex.y())
        if N % 1000 == 0:
            print "Mean of primary vertices:", sumx/N,
            print "Beamspot:", beamspot.product().x0()
            
c = ROOT.TCanvas( "c", "c", 1200, 800)
vtx_xy.Draw("colz")
c.SaveAs("vtx_xy.png")
~~~
{: .language-python}
Add the analogous 2D plots for x versus z and y vs z positions.
> ## More exercises...
> Furthermore, you can add a plot of the average primary vertex position and compare it to:
> *	the center of beamspot (red line)
> *	the beamspot region (beamspot center +/- beamspot witdh, that you can access through `beamspot.product().BeamWidthX()`
{: .challenge}
> ## Answer
> ~~~
> import DataFormats.FWLite as fwlite
> import math
> import ROOT
> from array import array
> 
> ROOT.gROOT.SetBatch(True)
> 
> def isGoodPV(vertex):
>     if ( vertex.isFake()        or \
>          vertex.ndof < 4.0      or \
>          abs(vertex.z()) > 24.0 or \
>          abs(vertex.position().Rho()) > 2):
>            return False
>     return True
> 
> events          = fwlite.Events("file:run321167_ZeroBias_AOD.root")
> primaryVertices = fwlite.Handle("std::vector<reco::Vertex>")
> beamspot        = fwlite.Handle("reco::BeamSpot")
> vtx_position, N_vtx = array( 'd' ), array( 'd' )
> 
> vtx_xy = ROOT.TH2F('vtx_xy','; x [cm]; y [cm]', 100,-0.05, 0.25, 100, -0.2, 0.1)
> c = ROOT.TCanvas( "c", "c", 1200, 800)
> 
> sumx = 0.0
> N    = 0
> iIOV = 0
> last_beamspot = None
> last_beamspot_sigma = None
> 
> vtx_position, N_vtx = array( 'd' ), array( 'd' )
> leg = ROOT.TLegend(0.45, 0.15, 0.6, 0.28)
> leg.SetBorderSize(0)
> leg.SetTextSize(0.03)
> 
> events.toBegin()
> for event in events:
>     event.getByLabel("offlinePrimaryVertices", primaryVertices)
>     event.getByLabel("offlineBeamSpot", beamspot)
> 
>     if last_beamspot == None or last_beamspot != beamspot.product().x0():
>         print "New beamspot IOV (interval of validity)..."
> 
>         ## first save tgraph and then reset
>         if (iIOV > 0):
>             theGraph   = ROOT.TGraph(len(vtx_position), N_vtx, vtx_position)
>             theGraph.SetMarkerStyle(8)
>             theGraph.SetTitle('IOV %s; N Vtx; X position'%iIOV)
>             theGraph.Draw('AP')
>             theGraph.GetYaxis().SetRangeUser(0.094, 0.099)
>             
>             line = ROOT.TLine(0,last_beamspot,N_vtx[-1],last_beamspot)
>             line.SetLineColor(ROOT.kRed)
>             line.SetLineWidth(2)
>             line.Draw()
> 
>             line2 = ROOT.TLine(0,last_beamspot - last_beamspot_sigma,N_vtx[-1],last_beamspot - last_beamspot_sigma)
>             line2.SetLineColor(ROOT.kOrange)
>             line3 = ROOT.TLine(0,last_beamspot + last_beamspot_sigma,N_vtx[-1],last_beamspot + last_beamspot_sigma)
>             line3.SetLineColor(ROOT.kOrange)
>             line2.SetLineWidth(2)
>             line3.SetLineWidth(2)
>             line2.Draw()
>             line3.Draw()
>             
>             leg.Clear()
>             leg.AddEntry(theGraph,  'ave. vtx position',        'p')
>             leg.AddEntry(line    ,  'center of the beamspot ' , 'l')
>             leg.AddEntry(line2   ,  'center of the bs #pm beamspot width ' , 'l')
>             leg.Draw()
>             c.SaveAs("vtx_x_vs_N_%s.png"%iIOV)
>             break
> 
>             vtx_position, N_vtx = array( 'd' ), array( 'd' )
> 
>         last_beamspot       = beamspot.product().x0()
>         last_beamspot_sigma = beamspot.product().BeamWidthX()
>         sumx = 0.0
>         N = 0
>         iIOV += 1
> 
>     for vertex in primaryVertices.product():
>         if not isGoodPV(vertex):  continue
>         N += 1
>         sumx += vertex.x()
>         if N % 1000 == 0:
>             vtx_position.append(sumx/N)
>             N_vtx.append(N)
> ~~~
> {: .language-python}
{: .solution}


## Primary vertices improve physics results

Finally, let's consider an example of how primary vertices are useful to an analyst. If you're interested in K<sub>S</sub> → π<sup>+</sup>π<sup>-</sup>, it might seem that primary vertices are irrelevant because neither of your two visible tracks (π<sup>+</sup>π<sup>-</sup>) directly originated in any of the proton-proton collisions. However, the K<sub>S</sub> did. The K<sub>S</sub> flew several centimeters away from the primary vertex in which it was produced, and its direction of flight must be parallel to its momentum (by definition). 

We can measure the K<sub>S</sub> momentum from the momenta of its decay products, and we can identify the start and end positions of the K<sub>S</sub> flight from the locations of the primary and secondary vertices. The momentum vector and the displacement vector must be parallel. (There is no constraint on the length of the displacement vector because the lifetimes of K<sub>S</sub> mesons follow an exponentially random distribution.)


Create a new file `analyse.py` in `TrackingShortExercize/` which will open `output.root`:
~~~
import math
import DataFormats.FWLite as fwlite
import ROOT

events = fwlite.Events("file:output.root")
primaryVertices = fwlite.Handle("std::vector<reco::Vertex>")
secondaryVertices = fwlite.Handle("std::vector<reco::VertexCompositeCandidate>")
~~~
{: .language-python}

Create a histogram in which to plot the distribution of `cosAngle`, which is the normalized dot product of the K<sub>S</sub> momentum vector and its displacement vector. Actually, make two histograms to zoom into the `cosAngle → 1` region.
~~~
cosAngle_histogram = ROOT.TH1F("cosAngle", "cosAngle", 100, -1.0, 1.0)
cosAngle_zoom_histogram = ROOT.TH1F("cosAngle_zoom", "cosAngle_zoom", 100, 0.99, 1.0)
~~~
{: .language-python}
> ## Question 9
**Construct a nested loop over secondary and primary vertices to compute displacement vectors and compare them with K<sub>S</sub> momentum vectors.** Just print out a few values.
{: .challenge}
> ## Answer
> ~~~
> for i, event in enumerate(events):
>     event.getByLabel("offlinePrimaryVertices", primaryVertices)
>     event.getByLabel("SecondaryVerticesFromLooseTracks", "Kshort", secondaryVertices)
>     for secondary in secondaryVertices.product():
>         px = secondary.px()
>         py = secondary.py()
>         pz = secondary.pz()
>         p = secondary.p()
>         for primary in primaryVertices.product():
>             dx = secondary.vx() - primary.x()
>             dy = secondary.vy() - primary.y()
>             dz = secondary.vz() - primary.z()
>             dl = math.sqrt(dx**2 + dy**2 + dz**2)
>             print "Normalized momentum:", px/p, py/p, pz/p,
>             print "Normalized displacement:", dx/dl, dy/dl, dz/dl
>     if i > 20: break
> ~~~
> {: .language-python}
{: .solution}

For each set of primary vertices, find the best cosAngle and fill the histograms with that.
> ## Solution
> ~~~
> events.toBegin()
> for event in events:
>     event.getByLabel("offlinePrimaryVertices", primaryVertices)
>     event.getByLabel("SecondaryVerticesFromLooseTracks", "Kshort", secondaryVertices)
>     for secondary in secondaryVertices.product():
>         px = secondary.px()
>         py = secondary.py()
>         pz = secondary.pz()
>         p = secondary.p()
>         bestCosAngle = -1       # start with the worst possible
>         for primary in primaryVertices.product():
>             dx = secondary.vx() - primary.x()
>             dy = secondary.vy() - primary.y()
>             dz = secondary.vz() - primary.z()
>             dl = math.sqrt(dx**2 + dy**2 + dz**2)
>             dotProduct = px*dx + py*dy + pz*dz
>             cosAngle = dotProduct / p / dl
>             if cosAngle > bestCosAngle:
>                 bestCosAngle = cosAngle # update it if you've found a better one
>         cosAngle_histogram.Fill(bestCosAngle)
>         cosAngle_zoom_histogram.Fill(bestCosAngle)
> 
> c = ROOT.TCanvas("c" , "c" , 800, 800)
> cosAngle_histogram.Draw()
> c.SaveAs("cosAngle.png")
> cosAngle_zoom_histogram.Draw()
> c.SaveAs("cosAngle_zoom.png")
> ~~~
> {: .language-python}
{: .solution}
Finally, create K<sub>S</sub> mass histograms, with and without requiring `bestCosAngle` to be greater than `0.99`. Does this improve the signal-to-background ratio?
> ## Solution
> ~~~
> mass_histogram = ROOT.TH1F("mass", "mass", 100, 0.4, 0.6)
> mass_goodCosAngle = ROOT.TH1F("mass_goodCosAngle", "mass_goodCosAngle", 100, 0.4, 0.6)
> 
> events.toBegin()
> for event in events:
>     event.getByLabel("offlinePrimaryVertices", primaryVertices)
>     event.getByLabel("SecondaryVerticesFromLooseTracks", "Kshort", secondaryVertices)
>     for secondary in secondaryVertices.product():
>         px = secondary.px()
>         py = secondary.py()
>         pz = secondary.pz()
>         p = secondary.p()
>         bestCosAngle = -1       # start with the worst possible
>         for primary in primaryVertices.product():
>             dx = secondary.vx() - primary.x()
>             dy = secondary.vy() - primary.y()
>             dz = secondary.vz() - primary.z()
>             dl = math.sqrt(dx**2 + dy**2 + dz**2)
>             dotProduct = px*dx + py*dy + pz*dz
>             cosAngle = dotProduct / p / dl
>             if cosAngle > bestCosAngle:
>                 bestCosAngle = cosAngle # update it if you've found a better one
>         cosAngle_histogram.Fill(bestCosAngle)
>         cosAngle_zoom_histogram.Fill(bestCosAngle)
>         if bestCosAngle > 0.99:
>             mass_goodCosAngle.Fill(secondary.mass())
>         mass_histogram.Fill(secondary.mass())
> 
> c = ROOT.TCanvas("c", "c", 800, 800)
> mass_histogram.Draw()
> mass_goodCosAngle.SetLineColor(ROOT.kRed)
> mass_goodCosAngle.Draw("same")
> c.SaveAs("mass_improved.png")
> ~~~
> {: .language-python}
{: .solution}

**That's all!** The session is over, unless you would like to try some of the extra questions and arguments listed in the [Appendix](https://bdanzi.github.io/trackingvertexing/Appendix/index.html)!
{% include links.md %}

