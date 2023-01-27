Host Info:

2016 Lenovo Ideapad

Intel Pentium 4405U (2C/4T)

8GB DDR3

Ubuntu 22.04.1 LTS

I followed the, "The Slightly Tricky Way", part of [this guide](https://all3dp.com/2/octoprint-linux-ubuntu-tutorial/) published by ALL3DP. After doing most (I just realized now, typing this, that the tutorial does in fact mention adding the user to the correct groups. That'll be relevant later, woops!\*\*) of those steps I noticed that my printer wasn't showing up in the devices list.

Seeing this I took a look at dmesg to see what it was doing:

`sudo dmesg | grep tty`

It turns out that a plugin, 'brltty', was causing my printer to connect, register on the bus, then disconnect. I found a solution [here](https://stackoverflow.com/questions/26410562/why-usb-device-gets-disconnected-immediately-after-successful-probe), which was to uninstall brltty:

`sudo apt remove brltty`

After that I rebooted the machine, thinking surely it'd be a quick connect and off to the printing wonderland. Unfortunately this wasn't the case; Octoprint saw the printer in the usb port, but could not establish a connection. I again checked dmesg to see if the printer had been recognized, thankfully it was. Yet octoprint would fail on auto detect, but hang when I selected a baud rate. It turns out it was just a permissions issue!\*\* During the steps to install octoprint, a new user, octo, was created with minimal permissions. In order to allow 'octo' to read the USB devices and access them it must be added to the 'dialout' group via:

`sudo adduser octo dialout`

After doing that the auto connect worked! I got my KG to connect automatically with a baud rate of 115200.

One of the awesome features of octoprint is camera monitoring, which the laptop has built in via the webcam. Unfortunately, octoprint doesn't support native cameras but instead supports viewing a stream from some camera. This is where things get a little more interesting, so I'm going to move to step-by-step:

1. Login to your host as the normal user (not octo).
2. Install motion:`apt install motion`
3. Set motion to be started automatically on boot:`sudo systemctl start motion`
4. Restart your host.
5. Find where motion is hiding its master config file, normally it is in:`/etc/motion/motion.conf`
6. Edit the config file with sudo privilege:`sudo nano motion.conf`
7. In the file find the following lines and change them:
   1. move\_output, on or off, toggles writing video clips to your host.`movie_output off`
   2. stream\_port specifies the port that motion will use for the stream, can be any integer`stream_port 1566`
   3. stream\_localhost sets if other computers on the local network can see the stream, off for public on for private`stream_localhost off`
   4. stream\_maxrate sets the frame-rate for the stream.`steam_maxrate 30`
   5. stream\_quality is pretty self explanatory, it goes from 0 to 100.`stream_quality 50`
   6. stream\_preview\_method look to the [documentation](https://motion-project.github.io/motion_config.html#stream_preview_method) to decide on this one.`stream_preview_method 4`
8. Save this file by pressing Control+x
9. Restart the motion service:`sudo systemctl restart motion`
10. In a web browser type in the IP of your machine and the port specified in the config file, as an example: [192.168.53.234:1566](https://192.168.53.234:1566)
11. The stream should now show up! Full motion, full color video!
12. Now we can go to the octoprint settings, to 'Webcam & Timelapse", and enter the previous URL into the 'Stream URL' section.
13. Hit test and watch!

So thats how I went about using an old laptop and integrated webcam to monitor my prints with octoprint!

If there are any issues or questions do let me know! I'll try my best to answer and correct things as needed. Hoping to contribute more to the community in the future, specifically with the Go :-)

Future ambitions include an extruder upgrade and motherboard swap! Those will also be posted here, pass or fail lol
