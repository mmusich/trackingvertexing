---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
venue: "LHC Physics Center, Fermi National Accelerator Laboratory"        # brief name of the institution that hosts the workshop without address (e.g., "Euphoric State University")
address: "In Person"      # full street address of workshop (e.g., "Room A, 123 Forth Street, Blimingen, Euphoria"), videoconferencing URL, or 'online'
country: "us"      # lowercase two-letter ISO country code such as "fr" (see https://en.wikipedia.org/wiki/ISO_3166-1#Current_codes) for the institution that hosts the workshop
language: "en"     # lowercase two-letter ISO language code such as "fr" (see https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) for the
latitude: "41.842258"        # decimal latitude of workshop venue (use https://www.latlong.net/)
longitude: "-88.245781"       # decimal longitude of the workshop venue (use https://www.latlong.net)
humandate: "Synchronously January, 2023"    # human-readable dates for the workshop (e.g., "Feb 17-18, 2020")
humantime: "24/7"    # human-readable times for the workshop (e.g., "9:00 am - 4:30 pm")
startdate: 2023-01-09      # machine-readable start date for the workshop in YYYY-MM-DD format like 2015-01-01
enddate: 2023-01-13        # machine-readable end date for the workshop in YYYY-MM-DD format like 2015-01-02
email: ["brunella.d'anzi@cern.ch"]
---

![CMS Detector Slice](https://cmsexperiment.web.cern.ch/sites/cmsexperiment.web.cern.ch/files/detectoroverview.gif){:width="50%"}

We will present an introduction to using tracks for analyses in the era of **large pile-up** (many primary vertices). Our exercises will all use real data and will familiarize you with the following techniques:

- Extracting **basic track parameters** and reconstructing invariant masses from tracks in CMSSW. Tracks are the detector entities that are closest to the four-vectors of particles: the momentum of a track is nearly the momentum of the charged particle itself.
- Cleaning **sets of tracks for analysis**. We will use filters to eliminate bad tracks and discuss sources of tracking uncertainties. These filters are provided by the tracking POG (Physics Object Group)
- Extracting **basic parameters of primary vertices**. In this high-luminosity era, it is not uncommon for a single event to contain as many as thirty to fifty independent collisions. For most analyses, only one is relevant, and it can usually be identified by its tracks.
- Using tracks to measure from data and MC the **tracking efficiency** by using tag and probe method via dimuon resonances. It can be extended to measure the efficiency of any selection of muons (trigger, isolation, identification) and derive scale factors from MC.
<!-- this is an html comment -->

<h2 id="general">General Information</h2>

{% if site.carpentry == "swc" %}
{% include swc/intro.html %}
{% elsif site.carpentry == "dc" %}
{% include dc/intro.html %}
{% elsif site.carpentry == "lc" %}
{% include lc/intro.html %}
{% endif %}

<p id="what">
  <strong>What:</strong> A series of pre-exercises to exercise all the needed tools with the laptop participants will bring to CMS DAS, so that they can be ready to go from the beginning of the school.
</p>

{% comment %}
AUDIENCE

Explain who your audience is.  (In particular, tell readers if the
workshop is only open to people from a particular institution.
{% endcomment %}
{% if site.carpentry == "swc" %}
{% include swc/who.html %}
{% elsif site.carpentry == "dc" %}
{% include dc/who.html %}
{% elsif site.carpentry == "lc" %}
{% include lc/who.html %}
{% endif %}

{% comment %}
LOCATION

This block displays the address and links to maps showing directions
if the latitude and longitude of the workshop have been set.  You
can use https://itouchmap.com/latlong.html to find the lat/long of an
address.
{% endcomment %}
{% assign begin_address = page.address | slice: 0, 4 | downcase  %}
{% if page.address == "online" %}
{% assign online = "true_private" %}
{% elsif begin_address contains "http" %}
{% assign online = "true_public" %}
{% else %}
{% assign online = "false" %}
{% endif %}
{% if page.latitude and page.longitude and online == "false" %}
<p id="where">
  <strong>Where:</strong>
  {{page.address}}.
  Get directions with
  <a href="//www.openstreetmap.org/?mlat={{page.latitude}}&mlon={{page.longitude}}&zoom=16">OpenStreetMap</a>
  or
  <a href="//maps.google.com/maps?q={{page.latitude}},{{page.longitude}}">Google Maps</a>.
</p>
{% elsif online == "true_public" %}
<p id="where">
  <strong>Where:</strong>
  online at <a href="{{page.address}}">{{page.address}}</a>.
  If you need a password or other information to access the training,
  the instructor will pass it on to you before the workshop.
</p>
{% elsif online == "true_private" %}
<p id="where">
  <strong>Where:</strong> This training will take place online.
</p>
{% endif %}

{% comment %}
DATE

This block displays the date and links to Google Calendar.
{% endcomment %}
{% if page.humandate %}
<p id="when">
  <strong>When:</strong>
  {{page.humandate}}.
  <!--{% include workshop_calendar.html %}-->
</p>
{% endif %}

{% comment %}
SPECIAL REQUIREMENTS

Modify the block below if there are any special requirements.
{% endcomment %}
<p id="requirements">
  <strong>Requirements:</strong> Participants must have access to a computer with internet access for which they have administrative privileges. Acceptable operating systems include Mac OS, Linux, or Windows (preferably not a tablet, Chromebook, etc.). The <a href="setup.html">setup</a> page will have more information about any additional pieces of software that must be installed or any accounts which must be obtained.
</p>

{% comment %}
ACCESSIBILITY

Modify the block below if there are any barriers to accessibility or
special instructions.
{% endcomment %}
<p id="accessibility">
  <strong>Accessibility:</strong>
{% if online == "false" %}
  We are committed to making this workshop
  accessible to everybody. The workshop organizers have checked that:
</p>
<ul>
  <li>The room is wheelchair / scooter accessible.</li>
  <li>Accessible restrooms are available.</li>
</ul>
<p>
  Materials will be provided in advance of the workshop and
  large-print handouts are available if needed by notifying the
  organizers in advance.  If we can help making learning easier for
  you (e.g. sign-language interpreters, lactation facilities) please
  get in touch (using contact details below) and we will
  attempt to provide them.
</p>
{% else %}
  We are dedicated to providing a positive and accessible learning environment for all. Please
  notify the instructors in advance of the workshop if you require any accommodations or if there is
  anything we can do to make this workshop more accessible to you.
</p>
{% endif %}


<p id="instructors">
   <strong>Instructors:</strong>
   {% for name in page.instructor %}
   {% if forloop.last and page.instructor.size > 1 %}
   and
   {% else %}
   {% unless forloop.first %}
   ,
   {% endunless %}
   {% endif %}
   {{name}}
   {% endfor %}
</p>

<p id="helpers">
   <strong>Helpers:</strong>
   {% for name in page.helper %}
   {% if forloop.last and page.helper.size > 1 %}
   and
   {% else %}
   {% unless forloop.first %}
   ,
   {% endunless %}
   {% endif %}
   {{name}}
   {% endfor %}
</p>

{% comment %}
CONTACT EMAIL ADDRESS

Display the contact email address set in the configuration file.
{% endcomment %}
<p id="contact">
  <strong>Contact:</strong>
  Please email
  {% if page.email %}
  {% for email in page.email %}
  {% if forloop.last and page.email.size > 1 %}
  or
  {% else %}
  {% unless forloop.first %}
  ,
  {% endunless %}
  {% endif %}
  <a href='mailto:{{email}}'>{{email}}</a>
  {% endfor %}
  {% else %}
  to-be-announced
  {% endif %}
  for more information or assistance.
</p>

> ## Facilitators
> [Brunella D'Anzi*](https://twiki.cern.ch/twiki/bin/view/Main/BrunellaDAnzi) (University of Bari), [Caleb Smith](https://twiki.cern.ch/twiki/bin/view/Main/CalebJamesSmith) (Baylor), [Nicola De Filippis](https://twiki.cern.ch/twiki/bin/view/Main/NicolaDeFilippis) (Polytechnic of Bari), [Susan Dittmer](https://twiki.cern.ch/twiki/bin/view/Main/SusanDittmer) (University of Illinois Chicago)
*Lead Contact
{: .testimonial}

> ## Mattermost Chat
> The [Tracking and Vertexing Short Exercise](https://mattermost.web.cern.ch/cmsdaslpc2023/channels/shortextrackingvertexing) channel will be available once you join the [CMSDAS@LPC2023](https://mattermost.web.cern.ch/cmsdaslpc2023/channels/town-square) team. Direction for how to join this Mattermost chat team can be found on the <a href="setup.html">setup</a> page.
{: .discussion}

> ## Introduction Slides
> At the beginning of this short exercise an introduction will be done based on [these slides](https://twiki.cern.ch/twiki/pub/CMS/SWGuideCMSDataAnalysisSchoolLPC2023TrackingVertexingShortExercise/CMSDAS2023_TrackingVertexingExercise_Introduction.pdf)
{: .callout}
{% comment %} This is a comment in Liquid {% endcomment %}

> ## Prerequisites
> **Before going any further, please follow the instructions on the [setup page](setup.md).**
> [CMS DAS Pre-Exercises](https://fnallpc.github.io/cms-das-pre-exercises/)
{: .prereq}

{% include links.md %}
