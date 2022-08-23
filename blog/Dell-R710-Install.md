# Installing VMWare ESXi on a Dell PowerEdge R710

As many home lab enthusiasts have had before me, I had a problem: I needed more <b><i>POWAH</b></i>. From the 
virtual machines I was running for teaching and my own research/practice to the desire to offload some of those 
virtual machines from my desktop to free up its resources to the realization that for the services I was running, a 
cloud container platform may not be the most economically viable decision... I needed to beef up the private side of 
my hybrid cloud environment. While for enterprises, the predictable, scalable IaaS model works extremely well, that is 
not necessarily the case when it comes to a home lab or small business. The math for an enterprise may be (in a very 
basic sense) "It's cheaper to run a server for $300/month than it is to pay $10k for a new physical server," while 
the math for me is more "Do I want to spend $40/month for this cloud server or buy a 10-year-old enterprise server for 
less than $200?" I chose the latter. 

# Finding The Right Unit And Specs

As this was my first forray into purchasing enterprise IT equipment for myself, I wanted to make sure I took the time 
to figure out what I needed. Considering I intended from the start to use this as a virtualization server, deciding 
the compute, memory, and space requirements was a simple process of deciding how many machines I wanted to run on 
one server, what resources those machines needed, and then adding them together with a buffer for the server's OS 
overhead. After researching, I decided on the Dell PowerEdge R710 as its price point ($150-200 USD) more than 
compensated for some of the feature restrictions (like being restricted to ESXi 6.7) for my use case. I also ordered 
a 1 TB enterprise SSD, knowing full well that I would likely have to order another one in relatively short order. 

# Prepping My Environment

To be honest, there was not a ton of prep work to be done for this install. I already knew from the dimensions of the 
server that I was going to have to upgrade my server rack, as it only has a maximum depth of 24", while the R710 boasts 
a hefty 28" length, so I knew going into this that initial implementation would be resting on the top of my current 
rack while I waited for a new rack to arrive. I also needed a short ethernet cord to go from the R710's NIC to the 
switch, and two universal power cords for the redundant power supplies. Then, I realized I was going to hit a snag: 
the R710 was old enough that it only had a VGA output. Not only did I not have a monitor with VGA input, but I realized 
that without coming up with something better, I'd need to haul a monitor and keyboard to the server rack every time I 
wanted to log into the machine locally. That didn't seem feasible, but the world is full of home labbers, right? Surely 
someone had already solved this problem? Oh boy had they. 

What I discovered when I started searching for solutions to my problem was that someone had gone above and beyond just 
finding a workaround to get through an install. No no no. This was a solution that provided a quick and easy way for 
me to not just access my server, but to access and interact with anything in my house that had video output and USB input. 
I found the <b>TinyPilot</b>. 

TinyPilot is a KVM switch housed in a Raspberry Pi that runs in your browser, and it's <i>amazing</i>. While I chose 
to buy one from the inventor (both to support something that I think is a pinnacle example of 'necessity is the 
mother of invention' and because I was working with some time constraints), I definitely recommend you check out his 
tutorial [here](https://tinypilotkvm.com/blog/build-a-kvm-over-ip-under-100). However, if you want to support TinyPilot 
or if you are also working under time constraints, there's the [TinyPilot Voyager2](https://tinypilotkvm.com/product/tinypilot-voyager2). 

Next was definitely the easiest step, which was to download and install VMWare ESXi 6.7, which you can find 
[here](https://customerconnect.vmware.com/downloads/info/slug/datacenter_cloud_infrastructure/vmware_vsphere/6_7), 
and then to install the ESXi .iso on a USB drive using Rufus, which can be downloaded [here](https://rufus.ie/en/). 

# Installing ESXi 6.7

Finally, TinyPilot hooked up and ready to go, server "installed" on the rack (yes, I said on, not in), and an ESXi 
bootable USB in-hand, it was time to attempt the install. After working out the logistics of getting everything hooked 
up (the VGA to HDMI adapter I purchased with the TinyPilot is active, requiring USB power, I had the USB stick itself, 
then the TinyPilot also requires a USB connection to its host, and then the ethernet connections for both the TinyPilot 
and the server - and the USB stick I had wouldn't fit in the USB ports on the back of the server due to the housing on 
the USB stick, but luckily the TinyPilot's included USB-C to USB-A cable was <i>just</i> long enough), I powered on the 
R710 and was hit with my second issue: the server had been off for so long the RAID controller's battery had died. 

30 minutes later (20% waiting patiently for the RAID controller battery to charge and 80% freaking out because the RAID 
controller battery might be dead), and that hurdle was safely behind me, crisis averted. 

The next thing I ran into face-first was my own hubris. In my anticipation to get this thing working and in my frustration 
regarding how many times you have to hit "yes" to boot an operating system from time to time, I yes/continued right past 
the message saying that I had no RAID configuration. Once ESXi booted, guess what? I had no disks to install the OS on. 
Reboot, boot into the configuration manager, set up a quick RAID 0 (I only have one disk installed, so no worries to 
the people whos blood pressure just spiked), saved the config, and rebooted again. Back through the ESXi install 
prompts, a little more waiting while the installer does its thing, and then finally the message I'd been waiting for 
all night: <b>Please remove all bootable media and reboot to complete installtion</b>. 

After another reboot, I was met with the ESXi welcome screen. After a few quick configurations (setting a root password, 
changing the IP address from dynamic via DHCP to static (and then setting the subnet mask and default gateway), etc and 
one final reboot, I was able to navigate to the server's IP address in my browser and see the sweet, sweet login portal. 

![esxi-portal](https://i.redd.it/42x0xl8oanzy.png)

After logging in as the root user, checking to make sure all resources were reporting correctly, adding a DHCP 
reservation for the server, and adding a DNS record and load balancer content filter rule to access my ESXi server 
via a subdomain, I was ready to actually deploy a VM on the server! I snagged the .iso for Ubuntu 22.04 (I already 
had the .iso on my local machine, but if you need it, [here it is](https://releases.ubuntu.com/22.04/)), added it 
to the directory in ESXi, assigned resources, clicked "Create," and BOOM! Ubuntu VM running on a server in my private 
cloud. 

# Okay, Now I Have A Virtualization Server... What Do I Do With It?

For now, this will be used to run VMs that I don't want utilizing my host resources on my desktop but that need to run 
constantly (like my virtual load balancer from Kemp, <b><i>which is FREE and you should totally go get one right now</b></i>) 
as well as lab VMs for my students to both use and abuse as they learn offensive and defensive security techniques (or 
in the case of my Cyber Patriot students, exclusively defensive security with a touch of Cisco networking). When I don't 
need to have VMs running constantly, or when I eventually get another R710 or similar server, to move my container platform 
out of the public cloud and over to my private cloud to cut down on service expendiatures. In addition, you could use 
this server to build smaller virtual servers that you may need to host one service or another that you decide to host 
locally. In short, we now have a pool of local resources that we can turn into... well, almost anything!

# Wrap-Up and Lessons Learned

All in all, I had a blast setting this thing up and adding it to my environment. I definitely learned some things 
(like that I really do need to pay attention and not repeatedly smash the Enter key during boot, or that when picking 
a USB stick for the installer to pick one that isn't housed in a massive plastic shroud), I laughed, I cried... it 
was a good time. Not only can I not wait to get to play around with this thing some more, I can't wait to get another 
server and go through this process all over again!
