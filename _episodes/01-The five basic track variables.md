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
One of the oldest tricks in particle physics is to put a track-measuring device in a strong, roughly uniform magnetic field so that the tracks curve with a radius proportional to their momenta (see [derivation](http://en.wikipedia.org/wiki/Gyroradius#Relativistic_case)). Apart from energy loss and magnetic field inhomogeneities, the particles' trajectories are helices. This allows us to measure a dynamic property (momentum) from a geometric property (radius of curvature).

![Helical trajectory of a particle in a vertical magnetic field.](http://upload.wikimedia.org/wikipedia/commons/thumb/2/29/Helix.svg/410px-Helix.svg.png){:height="300"}
![Track parameters at LHC](https://www.lhc-closer.es/webapp/files/1435504163_ad6fd1cc4163a3a2d3c54388c80c45ba.jpg){:height="300"}
A helical trajectory can be expressed by five parameters, but the parameterization is not unique. Given one parameterization, we can always re-express the same trajectory in another parameterization. Many of the data fields in a CMSSW reco::Track are alternate ways of expressing the same thing, and there are functions for changing the reference point from which the parameters are expressed. (For a much more detailed description, see [this page](http://www-jlc.kek.jp/subg/offl/lib/docs/helix_manip/node3.html#SECTION00210000000000000000).)
~~~
print("Hello World")
~~~
{: .language-python}

{% include links.md %}

