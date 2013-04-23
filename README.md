Installation Instructions
==========
# PART 1, INSTALLING CutyCapt

# get CutyCapt, starting in /opt

cd /opt
# svn will create a subdirectory for CutyCapt

# need these libraries to get started
sudo apt-get install subversion libqt4-webkit libqt4-dev g++
sudo apt-get install subversion libapache2-svn
sudo apt-get install libqt4-core libqt4-dev libqt4-gui qt4-dev-tools
sudo apt-get install cpp gcc
sudo apt-get install build-essential g++

# get CutyCapt repository
sudo svn co https://cutycapt.svn.sourceforge.net/svnroot/cutycapt 

# cd to the directory containing the CutyCapt source
cd cutycapt/CutyCapt/
# create the Make file to compile CutyCapt
sudo qmake
sudo qmake-qt4

sudo gcc -v
sudo make -v
sudo make 

#SUCCESS!!

# to test CutyCapt, I need to install xvfb
sudo aptitude install xvfb
# test CutyCupt
xvfb-run --server-args="-screen 0, 1024x768x24" ./CutyCapt --url="http://www.learningware.com/" --out="lw.png"
ls #success!

# PART 2, INSTALLING and STARTING xvfb AS A SERVICE

# now I will install xvfb as a service
#  the complete instructions are at http://rynop.com/take-website-screenshots-on-headless-linux-di
# create the file in /etc/init.d 
sudo vi /etc/init.d/screencap

# paste in text below

----------FILE CONTENTS----------
#!/bin/sh

### BEGIN INIT INFO
# Provides:          screencap
# Required-Start:    $local_fs $remote_fs $network $syslog
# Required-Stop:     $local_fs $remote_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: starts the Xvfb server
# Description:       starts Xvfb using start-stop-daemon
### END INIT INFO

PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
DAEMON=/usr/bin/Xvfb
NAME=screencap
DESC=screencap
DAEMON_OPTS=":1 -screen 1 1024x768x24 -nolisten tcp"

test -x $DAEMON || exit 0

set -e

. /lib/lsb/init-functions

case "$1" in
  start)
		echo -n "Starting $DESC: "
		start-stop-daemon --start --make-pidfile --background --quiet --pidfile /var/run/$NAME.pid \
		    --exec $DAEMON -- $DAEMON_OPTS || true
		echo "$NAME."
		;;

	stop)
		echo -n "Stopping $DESC: "
		start-stop-daemon --stop --quiet --pidfile /var/run/$NAME.pid \
		    --exec $DAEMON || true
		echo "$NAME."
		;;

	restart|force-reload)
		echo -n "Restarting $DESC: "
		start-stop-daemon --stop --quiet --pidfile \
		    /var/run/$NAME.pid --exec $DAEMON || true
		sleep 1
		start-stop-daemon --start --make-pidfile --background --quiet --pidfile \
		    /var/run/$NAME.pid --exec $DAEMON -- $DAEMON_OPTS || true
		echo "$NAME."
		;;

	status)
		status_of_proc -p /var/run/$NAME.pid "$DAEMON" Xvfb && exit 0 || exit $?
		;;
	*)
		echo "Usage: $NAME {start|stop|restart|status}" >&2
		exit 1
		;;
esac

exit 0

-----------END FILE CONTENTS------------

# i have no idea what this does
sudo update-rc.d screencap defaults
# make it so screen cap can run, in my instance the owner is root, your situation may vary
sudo chmod 700 /etc/init.d/screencap
# start the service
sudo /etc/init.d/screencap start
#screencap server tends to go down, set cronjob to make sure it stays up

sudo crontab -e 

#add the following tasks

* * * * * sudo /etc/init.d/screencap start
* * * * * sudo rm -f /opt/tmp

# create tmp folder

sudo mkdir /opt/tmp
chmod 777 /opt/tmp

# go back to the CutyCapt directory and test it using the xvfb serice
cd /opt/cutycapt/CutyCapt/
DISPLAY=:1 ./CutyCapt --url="http://rynop.com" --out="rynop.jpg" --out-format=jpeg --plugins=off --delay=4000
ls #success!


# PART 3, INSTALLING the website_capture script, this is what I wrote

# cd to the directory where you wish to put the website_capture script

cd /opt
# get the project from the github

sudo git clone https://github.com/Zeega/webcapture.git

# cd down into the folder containing the script
cd webcapture

# create a link to the CutyCapt program, your location my vary
#  (a different approach would be to include CutyCapt in $PATH, but I did not write the script that way)
sudo ln -s ../cutycapt/CutyCapt/CutyCapt CutyCapt

# at this point, the contents of the webcapture dir should look like this
# (note that the x bit must be on for CutyCapt, aspect and webpage_capture)
>ls -la
total 28
drwxr-xr-x 3 root root 4096 Sep  1 02:30 .
drwxr-xr-x 4 root root 4096 Sep  1 02:30 ..
drwxr-xr-x 6 root root 4096 Sep  1 02:22 .svn
lrwxrwxrwx 1 root root   29 Sep  1 02:30 CutyCapt -> ../cutycapt/CutyCapt/CutyCapt
-rwxr-xr-x 1 root root 7590 Sep  1 02:22 aspect
-rwxr-xr-x 1 root root 4199 Sep  1 02:22 webpage_capture

# install imagemagick, required for the cropping, scaling and framing
apt-get install imagemagick --fix-missing

# PART 4, TESTING the website_capture script

# finally, test the script:
/opt/webcapture/webpage_capture -p 500x400 -t 200x200 -crop -frame -full "http://metalab.harvard.edu/"

# after a few seconds you will see output similar to the following
	thumbSize:200x200
	pageSize:500x400
	url:http://metalab.harvard.edu/
	outputDir:web_capture_files
	frame:web_capture_files/framedThumb_DMGxV5bU.png
	crop:web_capture_files/croppedThumb_DMGxV5bU.png
	full:web_capture_files/DMGxV5bU.png

# the last three lines of the above indicate the location of the files
#  which will be in a sub-directory "web_capture_files" of your current directory
dir web_capture_files/
	croppedThumb_DMGxV5bU.png  framedThumb_DMGxV5bU.png  DMGxV5bU.png


# to see the options for the script webpage_capture, use, without an arguments
/opt/webcapture/webpage_capture
	
	webpage_capture:
	USAGE: webpage_capture [-p widthxheight] [-t widthxheight] [-frame] [-crop] [-full] webURL [outputDir]
	
	OPTIONS:
	
	-p            widthxheight   input page size (width x height); value in pixels, default = 600x400
	-t            widthxheight   thumbnail size (width x height); value in pixels, default = 200x200
	
	
	Output options are -frame, -crop and -full.  You may specify any combination
	-frame        ouput a "framed" thumbnail
	-crop         ouput a "cropped" thumbnail
	-full         ouput a "full" capture, this is the default if neither -crop nor -full are specified
	
	outputDir     default is "web_capture_files"
	
	the program will echo the names of the files created, one per line, in the form
	  full:fileName_x
	  crop:fileName_y
	  frame:fileName_z


