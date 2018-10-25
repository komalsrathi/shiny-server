.. |date| date::

************
Shiny Server
************

:authors: Komal Rathi
:contact: rathik@email.chop.edu
:organization: DBHi, CHOP
:status: This is "work in progress"
:date: |date|

.. meta::
   :keywords: web, portal, rshiny, 2016
   :description: DBHi Rshiny Web Portal.


Set up Shiny Server:
--------------------

Upload all data to server
=========================

.. code-block:: bash

	scp -r marislab-webportal rathik@reslndbhicbio02.research.chop.edu:/home/rathik/shinyapps/neuroblastoma-web-portal/
	
	# change permissions so everyone can access it
	sudo chmod -R 777 /home/rathik/

Install R
=========

.. code-block:: bash

	sudo yum update
	sudo yum install R
	sudo yum install libcurl-devel openssl-devel

Change Rprofile
===============

.. code-block:: bash

	sudo vi /usr/lib64/R/library/base/R/Rprofile

	# add the following to the Rprofile
	# download method
	options(download.file.method = "libcurl")

	# default CRAN mirror
	local({
	  r <- getOption("repos")
	  r["CRAN"] <- "https://cran.rstudio.com/"
	  options(repos=r)
	})

Install Shiny
=============

.. code-block:: bash

	sudo su - -c "R -e \"install.packages('shiny')\""

Install Shiny Server
====================

.. code-block:: bash

	wget https://download3.rstudio.org/centos5.9/x86_64/shiny-server-1.4.4.801-rh5-x86_64.rpm
	sudo yum install --nogpgcheck shiny-server-1.4.4.801-rh5-x86_64.rpm

	# To manually start or stop the server, you can use the following commands.
	sudo systemctl start shiny-server
	sudo systemctl stop shiny-server
	sudo systemctl restart shiny-server
	sudo systemctl status shiny-server

	# Should shiny server should be run automatically at boot time
	sudo systemctl enable shiny-server
	sudo systemctl disable shiny-server

	# fetch all your data in corresponding app directory
	# do this before installing R packages
	sudo mount -o remount,exec /tmp 
	export TMPDIR=~/tmp # or do this. I had to do this after the latest upgrade.

	# now install packages
	sudo R

Edit config file to add your app
================================

.. code-block:: bash

	sudo vi /etc/shiny-server/shiny-server.conf

	# Instruct Shiny Server to run applications as the user "shiny"
	run_as shiny;
	preserve_logs true; # this is required to preserve logs

	# Define a server that listens on port 3838
	server {
	  listen 3838;

	  # Define a location at the base URL
	  location / {

	    # Host the directory of Shiny Apps stored in this directory
	    site_dir /srv/shiny-server;

	    # Log all Shiny output to files in this directory
	    log_dir /var/log/shiny-server;

	    # When a user visits the base URL rather than a particular application,
	    # an index of the applications available in this directory will be shown.
	    directory_index on;
	  }

	  # Define location at NWP URL
	  location /NWP {

	    # application directory
	    app_dir /home/rathik/shinyapps/marislab-webportal/;

	    # log directory
	    log_dir /home/rathik/shinyapps/marislab-webportal/logs;

	    # directory structure
	    directory_index on;
	  }
	}

Turn off firewall
=================

.. code-block:: bash

	sudo firewall-cmd --zone=public --add-port=3838/tcp --permanent && sudo firewall-cmd --zone=public --add-port=3838/tcp