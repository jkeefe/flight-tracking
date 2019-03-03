# Process notes for playing with flight tracker

## Gear

Bought: 

- Raspberry Pi 3
- SDR receiver: https://www.amazon.com/dp/B00P2UOU72/
- 16 GB SD card
- Power Cable

## Setup

Followed the Mac instructions here: https://flightaware.com/adsb/piaware/files/PiAwareforBeginners.pdf

Oops. Missed this part "You must also be sure you are selecting your extracted piaware-sd-card-3.1.0.img and not the
piaware-sd-card-3.1.0.img.zip." Assuming this is still true, decided to reflash. So had to move the `.img` file out of the extracted folder and into the downloads folder. 

Reflashed.

Taking this moment to mention that the Etcher program is awesome.

Had to go into Disk Utility to mount "piaware" so that I could edit `piaware-config.txt`.

The instructions are really great. And the link through to my pi via flightaware is pretty awesome.

## SSH configuration

Turns out you can't ssh into the piaware-configured pi by default. 

So I logged in using an HDMI monitor and USB keyboard using the default user `pi` and default password `flightaware`.

Changed the password using `passwd` so it wasn't the default.

Enabled ssh by going to the `/boot` directory and initializing a file called "ssh" using `touch ssh`.

Now I can ssh to the pi over my wifi! (I have to be on the same wifi network): `ssh pi@X.X.X.X` where X's are the ip address ... which flightaware tells you.

## Adding a feed to ADSB Exchange

Followed the "scripted setup" instructions here: https://www.adsbexchange.com/how-to-feed/

Be sure to set up an account at absd exchange first! https://www.adsbexchange.com/  You need that info during setup.

Doing the below kept getting an error that indicated it needed python, which isn't installed. So did:

`sudo apt-get install python`

```
sudo apt-get -y update
sudo apt-get upgrade
sudo apt-get -y install git
git clone https://github.com/jprochazka/adsb-exchange.git
cd adsb-exchange
chmod +x setup.sh
sudo ./setup.sh
cd ..
```

## Quartz Data - PI Setup

started with my existing piaware setup, which I built earlier. see https://github.com/jkeefe/flight-tracking/blob/master/PROCESS_NOTES.md (above)

took out SD card and edited the config file for the Quartz wifi

used AngryIP app to scan all the ports on the network to find the pi website: http://angryip.org/

found it at `10.0.1.29`

logged in using `ssh pi@10.0.1.29`

entered password

On the pi, start feeding the database with:

```
python dump1090-stream-parser.py --mysql-host=XXX.us-east-2.rds.amazonaws.com  --mysql-user==XXX --mysql-pass=XXX --mysql-database==XXX
```

Or, to keep it running when you log out (no hangup) and write to /dev/null so it doesn't make a growing nohup.out file ... using `nohup` ... `&` to keep script running after logout (no-hangup) (on the dump1090-stream-parser.py script):

```
cd dump1090-stream-parser

nohup python dump1090-stream-parser.py --client-id=1 --mysql-host=XXX.us-east-2.rds.amazonaws.com  --mysql-user==XXX --mysql-pass=XXX --mysql-database==XXX > /dev/null &
```

Making this run automatically on startup of the pi, with the help of [this blog post](https://rahulmahale.wordpress.com/2014/09/03/solved-running-cron-job-at-reboot-on-raspberry-pi-in-debianwheezy-and-raspbian/):

- log in to your pi using ssh
- switch to root user using `sudo bash`
- run the command `crontab -e`
- put your command as `@reboot bash /path/to/file/run.sh` save it and get back on terminal
- then start cron service by running `/etc/init.d/cron start`
- then one additional step is to edit the `/etc/rc.local` file and add the following line in `/etc/init.d/cron/start`  be sure that it should before `exit 0`.
- now reboot your system by command `reboot`

But my `@reboot` line is:

```
@reboot /usr/bin/python /home/pi/dump1090-stream-parser/dump1090-stream-parser.py --client-id=1 --mysql-host=XXX.us-east-2.rds.amazonaws.com  --mysql-user==XXX --mysql-pass=XXX --mysql-database==XXX > /dev/null &
```
