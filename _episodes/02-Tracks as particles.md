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

{% include links.md %}

