LOD2 Stack Config for health.data.gov development environment
___________________________________________________

This file is incomplete! USE AT YOUR OWN RISK - I'm working on it :)

This file provides the configuration procedure to install a virtual KDI-demo environment that is known to work on a Mac OS 10.7.2 host OS with Oracle/Sun VirtualBox 4.1.8 and the LOD2 stack at:

	http://socialdataweb.org/kdi/binaries/lod2stack-v1.0-ubuntu-10.10.vmdk

(NOTE: this .vmdk was originally made available from http://stack.lod2.eu/VirtualMachines/lod2stack-v1.0-ubuntu-10.10.vmdk - but over the course of creating this README they have since changed and now only provide a .rar file, presumably for use with VMWare Fusion/Player. I'll check out the .rar file there now with VMWare at some point in the very near future to see if/how this configuration might change.)

2011/12/29 george dot thomas 1 at hhs dot gov
___________________________________________________


Step 0: Setting up your VM with all the right software

REQUIRED:

Install VirtualBox 4.1.8 and fire it up. 
Create a new 4096MB Linux/Ubuntu VM and 'use existing' disk, pointing to the location of your downloaded lod2stack-v1.0-ubuntu-10.10.vmdk file. 
Start your VM, click OK or hit return for any startup prompts. Finally, log in as the lod2 user with pwd lod2. 

In the VirtualBox / Devices menu, select 'Install Guest Additions' and when the VM Ubuntu instance responds, say 'OK' and 'Run', giving it the lod2 pwd when asked and click 'Authenticate'. Restart Ubuntu in your VM and log in again as lod2/lod2 - you're now in hi-res video mode, so you can select View / Switch to Seamless Mode, which you'll want to do. 

Update the Virtuoso binaries by downloading 6.1.4 at: 

	ftp://download.openlinksw.com/support/hugh/virt-614.tar.gz
	(or http://socialdataweb.org/kdi/binaries/virt-614.tar.gz if the ftp site/file is unavailable)

To install the components;

Shutdown the current Virtuoso instance with the command
	
	sudo /etc/init.d/virtuoso-opensource-6.1 stop

Extract the contents of the archive in a temp location of your choice, then from the extracted “virt-614” directory copy the following file to the location indicated:

	sudo cp virtuoso-t /usr/bin
	sudo cp *.vad /usr/share/virtuoso/vad

Restart the Virtuoso Server with the command:

	sudo /etc/init.d/virtuoso-opensource-6.1 start

That should be it, as the updated VAD application should be installed /upgraded automatically when the server starts. Check from the “System Admin” -> “Packages” tab of the Virtuoso Conductor that the VADs have been updated, if any have not then you can manually install from the Conductor.

Check the binary version which should be:

	lod2@ubuntu:~/binaries$ virtuoso-t -?
	Virtuoso Open Source Edition (multi threaded)
	Version 6.1.4.3127-pthreads as of Dec 10 2011
	Compiled for Linux (i686-pc-linux-gnu)
	Copyright (C) 1998-2011 OpenLink Software 
	[...]

OPTIONAL: (SKIP AHEAD TO STEP 1 if you want, or - ) Install Google Chrome browser (it's a lot faster than the version of Firefox that Ubuntu 10.10 comes with and easy to install/use), Google Refine with DERI RDF extensions (for transforming from tabular.xls to graph.ttl files), Top Braid Composer (an IDE for RDF/RDFS/OWL), the Siren/Solr search engine (for indexing triples using N-Triples based RDF syntax), and cURL (for testing Linked Data serving). 

Fire up Firefox and download/install a 32bit deb of Chrome at google.com/chrome, and TopQuadrant's Top Braid Composer Free Edition at http://topquadrant.com/products/TB_Composer.html by selecting 'Try', then look at the bottom of the page and click on the 'Install' link, then download the a 32bit of the free version for Linux. 

Before installing TBC-FE, you need to install the Sun/Oracle JREv6. Go to Ubuntu System / Administration and fire up the Synaptic Package Manager, logging in with lod2 pwd. Search for java, then install the sun-java6-bin and sun-java6-jre (if they aren't installed already – TBC won't run on OpenJDK). Tell Ubuntu which JRE to use by running;

	sudo update-alternatives --config java

and select the java-6-sun/jre over the OpenJDK version, then fire up TBC-FE from /home/lod2/tbcfree. 

Download Google Refine at http://code.google.com/p/google-refine/wiki/Downloads and install according to http://code.google.com/p/google-refine/wiki/InstallationInstructions, then download and install the DERI RDF extension for Google Refine at http://lab.linkeddata.deri.ie/2010/grefine-rdf-extension/ and install according to instructions given there. (Extract Refine 2.5 into /home/lod2, then extract the DERI RDF 0.7 archive into /home/lod2/google-refine-2.5/webapp/extensions). Fire up Google Refine + DERI RDF extensions with ./refine and http://127.0.0.1:3333/ will open. Unfortunately the 'RDF Schema Alignment' html view for the RDF plugin's 'Edit RDF Skeleton' feature renders horribly (with Firefox or Chrome) even though it works, so you may opt for using Refine with a browser running on a different OS (than Ubuntu 10.10).  

cURL:

	sudo apt-get install curl




***********************


NOTE - if/when Update Manager suggests a bunch of Ubuntu updates, just say no! 


Step 1: Reconfigure Apache and Virtuoso to accommodate multple virtual hosts

Stop Apache2
	
	sudo /etc/init.d/apache2 stop

Goto

	http://localhost:8890/conductor

and log in with dba/dba unap

Click on Web Admin GUI Tabs -

	System Admin / Parameters / HTTPServer

and change 'ServerPort' to 80.

	sudo gedit /etc/hosts

and add 

	127.0.0.1		health.data.gov
	127.0.0.1		vocab.data.gov
	127.0.0.1		reference.data.gov
	127.0.0.1		health.socialdataweb.gov

to the file and save it. 

	sudo gedit /etc/apache2/ports.conf 

file and change:

	NameVirtualHost *:80
	Listen 80

	to 

	NameVirtualHost *:8090
	Listen 8090

Restart Ubuntu on your VM

Visit

	http://health.data.gov:8090/ontowiki/ 

for your local VM's Virtuoso backed Ontowiki installation, and

	http://localhost/conductor 

for your local Virtuoso web admin console. 

Log into http://localhost/conductor using dba/dba unap
 
Under Conductor / System Admin / Packages

check to see if 'fct' vad 1.10 is installed, if not, click 'Install' over on the right of it's row in the conductor UI and proceed to install. 

Under System Admin / Parameters / URIQA UI tabs, set DynamicLocal to '0' and DefaultHost to 'health.data.gov', and click save.

Install the Virtuoso URL rewriting rules and syntax transformation stored procs for health.data.gov, reference.data.gov and vocab.data.gov using the KDI.sql file at 

	http://socialdataweb.org/kdi/configuration_files/KDI.sql

Use Virtuoso's ISQL Web UI, by selecting (clicking on) the 'Interactive SQL (ISQL)' link at the top left of the Virtuoso Conductor admin UI (at http://localhost/conductor), and a new browser window will open. 

	Copy the KDI.sql text into the form field and click 'Execute'. 

Restart Ubuntu on your VM


Step 2:

Download the following files from http://socialdataweb.org/lod2 to your local filesystem, somewhere accessible to the :

	bmm.owl
	codelist.rdfs
	drm.owl
	fea.owl
	govdata.rdfs
	healthcare.rdfs
	medicaid.rdfs
	medicare.rdfs

Create a TBC-FE project and import these files into that project for editing. 

Upload these metadata files into Virtuoso, by logging into http://localhost/conductor with dba/dba unap
In the Conductor Web UI, select Linked Data / Graph Store Upload tabs






