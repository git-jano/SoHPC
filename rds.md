In this short blog I would like to do something useful. My supervisor Gábor encouraged me to write a short Quick Start Guide to remote connections. The task we'll discuss is pretty common - you want to connect to another computer (assuming *NIX server) and see nice graphical output e.g. form 3D visualization app. Typical scenario in HPC is running visualizations on specialized nodes equipped with high performance graphics hardware and direct  connection to supercomputer's storage.

There are [many options](https://en.wikipedia.org/wiki/Comparison_of_remote_desktop_software). I will just briefly mention some commercial solutions, like common Microsoft [Remote Desktop Protocol](https://www.microsoft.com/en-US/store/Apps/Remote-Desktop/9WZDNCRFJ3PS) which works great connecting to Windows machines (form almost any platform), but lacks accelerated graphics on forks for other platforms servers. Well known [TeamViewer](https://www.teamviewer.com/) doesn't support 3D graphics at all, [Chrome Remote Desktop](https://chrome.google.com/webstore/detail/chrome-remote-desktop/gbchcmhmhahfdphkhkmpfmihenigjmpp) still needs developement and my liked [NoMachine](https://www.nomachine.com/) uses protocol to be described below.

We will concentrate on session forwarding  using VirtualGL and TurboVNC. 

##How does it work:
Normal X-forwarding sends all X rendering instructions and  the client does all the rendering on a CPU. VirtualGL overrides this protocol and renders 3D graphics at hosts GPU and then send only rendered 2D images to the client (the [scheme](http://www.virtualgl.org/About/Background) is much more complicated).
Or you can utilize this and render everything at host and transfer image of the desktop via VNC protocol. The Turbo stands for using turbo fast SIMD libraries for jpeg encoding.    


For more information and inspiration see links [1], [2], [3].

##How to run it example:

####Installing VGL on remote server####Installing VGL on remote server
on the remote visualization station  (needs root):

 1. get and install packages [VirtualGL](http://sourceforge.net/projects/virtualgl/)  and [TurboVNC](http://sourceforge.net/projects/turbovnc/files) from SourceForge 
 2. configure the VGL server and modify the X-server trough command line
	```bash 
	sudo vglserver_config
	```
 
 3. restart display manager (xdm or lightdm) or just reboot server 
	```bash
	sudo /etc/init.d/xdm restart 
	```
####Installing on your local client
On your local client install the very same packages  (needs root):
 1. Get and install packages [VirtualGL](http://sourceforge.net/projects/virtualgl/)  and [TurboVNC](http://sourceforge.net/projects/turbovnc/files) from SourceForge .
 
###Scenario A - Making VirtualGL connection 
 This is the easiest choice if you want to run only certain application remotely. 
 
 1. Establish connection (type in terminal)
	```bash
	vglconnect -s <user@graphics-server> 
	```
	
	e.g. ```vglconnect -s jano@vserver.debrecen.hpc.niif.hu``` 
	This will make connection with ssh-encrypted  X-forwarding and VGL Image transport. For more detail about options see [VirtualGL Reference](https://docs.oracle.com/cd/E19279-01/820-3257-12/VGL.html).
 2. Run your OpenGL application with ```vglrun``` command
	```bash
	vglrun application
	```
	
	e.g.  ```vglrun glxinfo | more``` to see OpenGL status or  ParaView ```vglrun paraview``` 
 
 ###Scenario B - Making TurboVNC connection 
 This is more complex (and a bit faster) solution for forwarding whole X session. More people can collaborate on the same session and it will last even after you close your connection, so you can continue with your work. 

	1. Start VNC server session on the remote visualization station by command 
	```bash
	vncserver
	```
	
		You will be prompted to set (and confirm) a password. If it was successful, you will obtain something like this 
	```
	Desktop 'TurboVNC: login:1 (jano)' started on display login:1
	Starting applications specified in /home/jano/.vnc/xstartup.turbovnc
	Log file is /home/jano/.vnc/login:1.log
	```
	
		the important here is the last number in the first line (after the colon) - the display number (```1``` in this case), by adding 5900 to it we'll obtain port number of the VNC (in this case ```1 + 5900 = 5901```).

		You can always check this by command ```vncserver -list```.
		You are allowed to disconnect now. 
	2. On your local machine create an ssh tunnel for encrypting  the VNC stream by typing 
		```bash
		 shh -N -q -L  1047:localhost:<display_number> user@server
		```
		where ```1047``` is arbitrary free port number and ```<display_number>``` is obtained from previous paragraph, e.g. ```ssh -N -q -L  1047:localhost:5902 jano@vserver.debrecen.hpc.niif.hu```.
	3. Now you can start TurboVNC client on your local machine  e.g. running command 
	```bash
	vncviewer localhost:1047
	```   
	 If you cannot find the ```vncviewer``` command, this might help  
	```
	/opt/TurboVNC/bin/vncviewer localhost:1047
	```
		Now you just need to type your chosen password for VNC and start X-session,
	4. Start 3D accelerated application with ```vglrun``` command
	```bash
	vglrun application
	```
		e.g.  ```vglrun glxinfo | more``` to see OpenGL status or  ParaView ```vglrun paraview```. (same as in the Scenario A)
	 
[1]:https://www.youtube.com/watch?v=_p-3_y6AcRAyour 
[2]:http://www.virtualgl.org/About/Background
[3]:https://en.wikipedia.org/wiki/VirtualGL