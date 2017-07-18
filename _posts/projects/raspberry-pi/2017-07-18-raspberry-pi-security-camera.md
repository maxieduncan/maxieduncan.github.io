---
layout: post
category : projects
title: "Raspberry Pi Security Camera"
tags : [raspberrypi, aws, iot, route53]
---
{% include JB/setup %}

I was looking at buying a security camera for home so I could monitor what was going on when I'm out and wasn't particularly taken with the options. There are plenty out there but most of the decent ones aren't cheap and require a subscription; not ideal as who knows how long these companies will actually stick around. Additionally, as this isn't something I need but simply would be nice to have, I wasn't interested in signing up for these services.

So I decided this would be a great Raspberry Pi project. A quick Google showed plenty of tutorials out there on how to set one up but I plan to add my own requirements. What I have in mind:
* Motion sensor to start/stop recording
* Ability to view camera feed remotely
* Switch to turn system on/off
* Link the motion sensor to Hue lights
* Look into turning the system on / off automatically as I leave / arrive home
* It could be fun to try and hook up some facial recognition

## Hardware
First up was getting the hardware:
* [Raspberry Pi 3](https://www.amazon.co.uk/gp/product/B01CD5VC92/ref=oh_aui_search_detailpage?ie=UTF8&psc=1)
* [Camera module](https://www.amazon.co.uk/gp/product/B01ER2SKFS/ref=oh_aui_search_detailpage?ie=UTF8&psc=1)
* Motion Sensor
* [Case](https://www.amazon.co.uk/gp/product/B01LZQS32Y/ref=oh_aui_search_detailpage?ie=UTF8&psc=1), the one I selected supported both the motion sensor and camera and came bundled with a motion sensor

The case I chose came in a flat pack that was a little finicky to put together and there were no instructions on how to connect the motion sensor. A quick google indicated where the connections were and that though they weren't labelled, if you got them the wrong way around the only outcome would be that the sensor didn't work. Luckily enough for me the first combination I chose was fine and the sensor worked as expect! The [Parent Detector](https://www.raspberrypi.org/learning/parent-detector/worksheet/) from raspberry pi gave me some simple code to check that the motion sensor and camera were both hooked up and working as expected.

## Software
Next up was the software. The Python code for motion detection and the camera seemed straightforward enough and the `motion` library looked like an easy way to get the camera feed up and running, so it looked like there would be a bit of worked involved but that it should be relatively straight forward.

Of course like most problems out there, there was already a solution ready to go, in this case [RPi Cam Web Interface](http://elinux.org/RPi-Cam-Web-Interface#Installation_Instructions). Now this doesn't do everything I'd like it to but it's customisable and out of the box provides all the functionality that I was looking to implement to start off. I don't like that it's an un-secure connection by default but you have a selection of http servers to choose from and configure as you see fit.

### Port Mapping
Once installed I had to configure a port mapping through my router to allow access to the outside world but once that was done I had an accessible feed that I could check in on remotely if I wanted to.

### Dynamic DNS
I don't have a static IP address, so I need a dynamic DNS service to keep external access available. I've used [No-IP](http://www.noip.com/) in the past which is a fine service and they do provide a Raspberry Pi client, however the free level means you need to sign in every 30 days to keep your address alive; which has been fine when I'm working on a project and just want to grant temporary access but what I really want here is something I can set up and forget.

To do this I decided to use Route53 as I've already got my main domain set up there. Of course there are plenty examples out there on how to do this, I've gone with a modified version of the [script provided by Will Warren](https://willwarren.com/2014/07/03/roll-dynamic-dns-service-using-amazon-route53/).

To use this I had to install the following libraries. I created an IAM user specifically for my Raspberry Pi device and gave them only Route53 access.

```
# Install the AWS CLI
sudo pip install awscli
# Then configure, I created a user for this especially in IAM
aws configure

# Need to install dnsutils for dig
sudo apt-get install dnsutils
```

My modified script just allows me to pass in the Zone ID and Record Set Name as arguments:

```
#!/bin/bash
#
# Modified version of the script provided by Will Warren at:
# https://willwarren.com/2014/07/03/roll-dynamic-dns-service-using-amazon-route53/
#

# Hosted Zone ID e.g. BJBK35SKMM9OE
ZONEID=$1

# The CNAME you want to update e.g. hello.example.com
RECORDSET=$2

if [ $# -eq 0 ]
  then
    echo "Usage route53.sh <Zone ID> <Record Set Name>"
    exit 1
fi

if [ -z "$1" ]
  then
    echo "No Zone ID supplied"
    exit 1
fi
if [ -z "$2" ]
  then
    echo "No Record Set Name supplied"
    exit 1
fi

# More advanced options below
# The Time-To-Live of this recordset
TTL=300
# Change this if you want
COMMENT="Auto updating @ `date`"
# Change to AAAA if using an IPv6 address
TYPE="A"

# Get the external IP address from OpenDNS (more reliable than other providers)
IP=`dig +short myip.opendns.com @resolver1.opendns.com`

function valid_ip()
{
    local  ip=$1
    local  stat=1

    if [[ $ip =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
        OIFS=$IFS
        IFS='.'
        ip=($ip)
        IFS=$OIFS
        [[ ${ip[0]} -le 255 && ${ip[1]} -le 255 \
            && ${ip[2]} -le 255 && ${ip[3]} -le 255 ]]
        stat=$?
    fi
    return $stat
}

# Get current dir
# (from http://stackoverflow.com/a/246128/920350)
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
LOGFILE="$DIR/update-route53.log"
IPFILE="$DIR/update-route53.ip"

if ! valid_ip $IP; then
    echo "Invalid IP address: $IP" >> "$LOGFILE"
    exit 1
fi

# Check if the IP has changed
if [ ! -f "$IPFILE" ]
    then
    touch "$IPFILE"
fi

if grep -Fxq "$IP" "$IPFILE"; then
    # code if found
    echo "IP is still $IP. Exiting" >> "$LOGFILE"
    exit 0
else
    echo "IP has changed to $IP" >> "$LOGFILE"
    # Fill a temp file with valid JSON
    TMPFILE=$(mktemp /tmp/temporary-file.XXXXXXXX)
    cat > ${TMPFILE} << EOF
    {
      "Comment":"$COMMENT",
      "Changes":[
        {
          "Action":"UPSERT",
          "ResourceRecordSet":{
            "ResourceRecords":[
              {
                "Value":"$IP"
              }
            ],
            "Name":"$RECORDSET",
            "Type":"$TYPE",
            "TTL":$TTL
          }
        }
      ]
    }
EOF

    # Update the Hosted Zone record
    aws route53 change-resource-record-sets \
        --hosted-zone-id $ZONEID \
        --change-batch file://"$TMPFILE" >> "$LOGFILE"
    echo "" >> "$LOGFILE"

    # Clean up
    rm $TMPFILE
fi

# All Done - cache the IP address for next time
echo "$IP" > "$IPFILE"

```

Then it was just a matter of setting up a cron job to run this regularly.

## Conclusion
Because of the work people have already done this was a lot easier to set up than I expected.

There are some limitations to this at the moment, any recordings will be stored on device itself (though they can be downloaded via the web interface) and I haven't hooked up the motion detection - by default this is provided by the camera feed rather than the motion detector.

It's also not an infrared camera so won't be much help at night which is why I'll look to connect the motion sensor to the hue lights at some stage.

But for now it does the job and I can look to add the more interesting features later.
