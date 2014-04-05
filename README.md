nweb README for version 23

1) Bug fixed - was duplicating errors in the nweb.log file 
   - thanks to Kieran Grant for stopping this and pointing out the fix.

2) Added support for favicon.ico - if nothing elase this will stop annoying 
	errors in the log file.  Most browsers on first encountering a webpage 
	also ask for this file.  The name mean favourite icon - Wikipedia has 
	more on this.  It is a tiny Bit Map image (normally called .bmp and 
	BMP editors can be used to create one - I used Windows Paint)
	I uses a very simple 16x16 pixels and 256 colour to keep the size and 
	complexity down.
	To add support I added .ico file extension to the allowed extensions 
	data structure and the sample favicon.ico file.
	This is then displyed by the browser at the little graphic next to 
	the URL. The one I made looks like this withe blue background.
	+---+
	|n  |
	|Web|
	+---+

3) As 1) removed a few lines, I managed to add 2) and still stay at 200 lines.


The code:
	nweb23.c -- the source code version 23
	client.c -- sample client end source code

nweb executable files for: 
	AIX 6.1 TL7 fnd AIX 7 TL1   POWER 
	Ubuntu 12.4  for x86_64     Intel/AMD 64 bit
	Ubuntu 12.4  for x86_32     Intel/AMD 32 bit
	Fedora 17 Linux for x86_64  Intel/AMD 64 bit
	OpenSUSE 12.1 for x86_64    Intel/AMD 64 bit
	Red Hat RHEL 6.3 for x86_64 Intel/AMD 64 bit
	SUSE SLES 11 for x86_64     Intel/AMD 64 bit
	Debian Squeeze for ARM on Raspberry Pi 

Minimum test website
	index.html  -- test very bascis web page 
	nigel.jpg   -- test image file
	favicon.ico -- the favourite icon bit map file


-----------------

nweb README for version 22

The nweb download file includes: 

README -- this file


This file contains lots of hints & tips on compiling and running the nweb code


nweb22.c
========
The 22 is the version number. 
nweb stands for Nigel's Web server but I some times call it a nano 
web server (i.e. very small).

This nweb is a web server that will respond to simple web browser 
requests for static files.
This version is exactly 200 lines long. 
I started with a target 100 lines and it worked fine too but then 
added comments, file type checks,
security checks, sensible directory checks and logging.

To compile this you need a basic C compiler use: cc nweb22.c -o nweb
If you want it to run faster you could include optimisation: 
cc -O2 nweb22.c -o nweb

Then to run it as root for the test files:
1) Place the index.html file and nigel.jpg in to a sensible directory 
	Note: not any system directories like / or /tmp 
	A director in /home would be a good idea - below we use /home/nigel/web

2) Make these files readable
	chmod ugo+r /home/nigel/web

3) Decide the port number
	Port number 80 is the web server default and assumed by web browsers.
	you might have to be the root user to use that  port number.
	If the machine is already running a web server then it will 
	probably be using port 80 so you can't us 80.
	I test using port 8181 but you have to make up your mind.

4) Start the nweb server:  /home/nigel/bin/nweb 80   /home/nigel/web
	or if using 8181   /home/nigel/bin/nweb 8181 /home/nigel/web

	If the program and files are all in /home/nigel/all
	You could use:
		cd /home/nigel/all
		rm -f nweb.log
		./nweb 80 .
		ps -ef | grep nweb 
		tail -f nweb.log

	See below for explanations for the rm, ps and tail

5) Note that any suitable files in the directory 2nd argument above 
could be served by the nmon web server - so don't have anything secret 
in that directory!

6) Use a browser to test it if using port 80 on a machine hostname 
of abc123.com browser to:
	http://abc123.com/index.html

   If using port 8181
	http://abc123.com:8181/index.html

7) Running nweb as a regualr user
You can run nweb as a regular user on some operating systems but not all.
I find using sudo works on some: 
	sudo /home/nigel/bin/nweb 80   /home/nigel/web

If you try and the log file reports socect connections errors worth 
	trying the 8181 port or higher numbers.  This is because 
	lower port numbers are reserved for root user use.

8) VERY IMPORTANT
nweb will behave like a daemon process, so it disconnects from the 
users command shell, goes into the back ground, closes input and 
output I/O & protects itself from you logging off.  
It will look like it stopped, when it is in fact still running in 
the background - check with ps -ef | grep nweb
Also note you will not see errors or warnings messages 
- they go in the log file.

Look in the nweb.log file for problems starting up or browsers connecting 
and requesting pages with: cat nweb.log
or: tail -f nweb.log

I find it good to remove the nweb.log before I start nweb or it is easy to get confused with previous error messages in the log.

9) As nweb runs as a daemon process it will try to run forever and not 
conntected to your user or terminal session.  Logging out will not 
effect it. To stop nweb you have to stop it by a KILL signal. 
Find the nweb process with: ps -ef | grep nweb
Then use: kill -9 PID 
to stop the nweb process.


10) nweb.log Log file - what to look for
Good example
Here is an edited down sample nweb.log file. 
Note the first line = "starting" looks good as there is no following error.

----
 INFO: nweb starting:8181:8913126
 INFO: request:GET /index.html HTTP/1.1 [[1KB of deleted request stuff here]]:2
 INFO: SEND:index.html:2
 INFO: Header:HTTP/1.1 200 OK
Server: nweb/21.0
Content-Length: 239
Connection: close
Content-Type: text/html

 INFO: request:GET /nigel.jpg HTTP/1.1  [[1KB of deleted request stuff here]]:3
 INFO: SEND:nigel.jpg:3
 INFO: Header:HTTP/1.1 200 OK
Server: nweb/21.0
Content-Length: 10184
Connection: close
Content-Type: image/jpg
---- 

Bad example
A failure to start due to the port number is not aallowed or in use looks like:

 INFO: nweb starting:80:8323296
ERROR: system call:bind Errno=13 exiting pid=8323296

----


client.c
========

This client.c program is designed to fake being a web browser.
It sends the expected requests to the web server over a network 
socket connection and displays the results as text rather than 
graphically displaying the results.  In the code you will have to 
change the two lines as below to match your web server or nweb server.

/* YOU WILL HAVE TO CHANGE THESE TWO LINES TO MATCH YOUR CONFIG */
#define PORT        8181		/* Port number as an integer - web server default is 80 */
#define IP_ADDRESS "192.168.0.8"	/* IP Address as a string */

The default is to request the /index.html from the web server. 
If you want to request another file then change the GET line as below:

char *command = "GET /index.html HTTP/1.0 \r\n\r\n" ;

To, for example:

char *command = "GET /nigel.jpg HTTP/1.0 \r\n\r\n" ;


Then compile the program with: cc client.c -o client

I save the output in to a file as putting a non-test file like .jpg to the 
terminal screen can cause chaos: client >output

Then edit the output file: vi output

In real life, the interaction of web browser and web server can be 
much more complex.
1) The web browser can tell the web server about its name, version 
	and capabilities.
2) The web server can send complex file types line JavaScript or Java 
	programs or other active components.
3) They can maintain a longer connection over the socket for efficiency.

4) Below is an example of my Firefox brower requesting an index.html file. 
I have added newline characters to make it readable - it is 1300 bytes long!
I have no idea what most of it is about. 
You will have to read the The World Wide Web Consortium (W3C) at 
http://www.w3.org for all the details.

GET /index.html HTTP/1.1**Host: myserver.home.com:80**User-Agent: Mozilla/5.0 
(W indows; U; Windows NT 5.1; en-GB; rv:1.9.2.28) Gecko/20120306 Firefox/3.6.28 
(.NET CLR 3.5.30729)**Ac cept: image/png,image/*;q=0.8,*/*;q=0.5**Accept-Language: 
en-gb,en;q=0.5**Accept-Encoding: gzip,defla te**Accept-Charset: ISO-8859-1,utf-8;
q=0.7,*;q=0.7**Keep-Alive: 115**Connection: keep-alive**Referer: 
http://myserver.uk.home.com:8181/index.html**Cookie: 
__utma=101107545.1790272076.1316019590.13289002 55.1328908680.164; 
__utmz=101107545.1328566199.157.46.utmcsr=t.co|utmccn=(referral)|utmcmd=referral|
u tmcct=/iTJx4DO1; UnicaNIODID=ZBr8gm79vIG-XKeoGGb; W3SSO_ACCESS=abc.home.com; 
ISP=70fdfc95 d93011d783e4de784ea97766-70fdfc95d93011d783e4de784ea97766-f67749a8b899e8ceed7e940b8c4bf189; 
Prof ile=2000121913394303111032836125|EN|866|866.BDF|en-GB; 
__unam=693fb60-1337f162b72-11770d11-5; WLS_ intra_USERID=nigel@hotmail.com; 
ipcInfo=cc%3Duk%3Blc%3Den%3Bac%3Dall; iwm1p=214617669; bprememberme=nigel@ hotmail.com; 
EPSPROFILE=EE2355DFE16AE020BE6C62FCB6BF5602; DWPERM=Xa.2/Xb.Xzso3-U35t8RWKvqBreGaQMgsP_RG 
Fl1124oIt-L-OPJIdSautkBN0D4NUp9JLlpUqPqB6CWOo-pgrJwhxNvvSfPAajgetaA2MOYwHfQPXPTRG9zwOMMR57EHQtXhOy5Om yzanyZthvVClm6uxvbwh0isEQ2Mm_9g2l7NjcA3RJdjuLaB3qlljOmyVuhDjBkgdNEb3PgYcCpbiu1FUzXrhPalhgsbAj7NBkaY88 Yyg/Xc./Xd./Xf./Xg.1696801 


I hope this has been instructive, thanks, Nigel Griffiths

