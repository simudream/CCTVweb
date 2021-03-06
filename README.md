CCTVweb
=======
Browse and manage MJPEG (motion JPEG) events from IQinvision IQeye IP Cameras stored via FTP through a web interface

Events are grouped together and individual events are loadable and can be replayed in real-time or by wheel scrolling. Both methods use Javascript rather than a video player.

Architecture
============
CCTVweb currently depends on the JPEG comment (COM) fields written by IQinvision IQeye series network cameras. Using a perl script, this data is read from those JPEGS, and is inserted into an RDBMS. A web frontend interfaces with a Python backend to offer a browser-based event viewer and an interface to run queries and set warnings and limits on cameras. Cameras are responsible for FTP'ing images, but no further intelligence is needed from them.

Requirements
============
* Linux server
* perl
  * exiftool
* python3
  * virtualenv
  * uwsgi
  * django
* mysql
* nginx

Installation
============
1. Get the most recent source:
  - git clone https://github.com/JayGoldberg/CCTVweb

### Python
1. Install Python, pip, virtualenv
2. Create virtualenv
  - virtualenv 

### Camera FTP configuration
1. Update your cameras to the latest firmware from [IQinvision](http://www.iqeye.com).
2. Under the Trigger tab, set up FTP to write images to filename "$ST.$FN", and directory "$SH/$SD(%Y)-$SD(%m)-$SD(%d)/$SD(%H)"
  -  See the table below for a description of these values

This will create file paths like "camera/00_34_33:e3:b4:81/2016-01-01/23/23_01.trig+00.jpg. It is important that this schema matches what is defined in config/config.py. Also, you will need to make sure that you define this schema to not write an excessive number of files to a single directory as this will impact performance.

#### IQinvision Dynamic Print (DP) variables
Variable | Description
------------- | -------------
$SH | Your camera’s hardware address.
$SI | Your camera’s IP address.
$SN | Your camera’s name, as specified on the Network Settings page.
$ST | The current time (in 24-hour format: HH:MM:SS, ex: 16:05:20).
$SD | The current date (ex: Wed Feb 03 2010).
$SC | Company name (e.g. IQinVision).
$SP | Product name (e.g. IQeye752).
$SV | The version of operating software on your camera.
$SM | The domain name, as specified on the Network Settings page.
$FN | The name of the file that your camera is accessing.
$MSE | Milliseconds since epoch
$IMGDBG | Image debug data
$O(oidNumber) | Display an OID, like IP address (3.6.10) or the image focus value(1.2.25) or last trigger event time (1.3.20). Use http://<yourcameraip>/oidtable.html to see all available OIDs.

For time-based variables, you can use $SD in combination with the common strftime() UNIX time variables.

License
=======
The MIT License (MIT), see http://opensource.org/licenses/MIT

Copyright (c) 2014 Jay Goldberg & contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

Roadmap
=======
* Write and commit the JPEG scraperto GitHub
* Design ORM in Django
* Design DB inserts from CSV generated by exiftool
* Write nginx/uWSGI backend for serving JPEGs to browser and running queries on DB
* Compile MJPEG events into H.264 for export and download
* Implement inotify on Linux to detect new events
* Implement warnings and alerts (disk full, excess events)
* Event viewer is JS-based, also has CSS overlays showing date/time/frame information, also display information in overlay related to where motion was triggered in the frame
* Queries such as:
  * All events triggered by motion in window 3, on 2014-06-05 on camera1, between 08:00-12:00
* Implement a multi-node infrastructure for cloud based deployments where there is a single nginx node and multiple uWSGI worker nodes
* Implement a live viewer that also monitors for event triggers in the camera stream a la http://netcameras.blogspot.com/2014/11/viewing-live-motion-data.html
