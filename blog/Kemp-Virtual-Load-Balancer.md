# Load Balancing My Domain And Network With The Kemp Load Balancer

As my homelab has grown, I have had an increasing and nagging suspicion that I needed to take a more active roll in 
handling the traffic that comes into my home lab. In addition, as part of my progress in my IT career, I knew I needed 
a website. While these were initially completely unrelated ideas, two videos by Network Chuck on YouTube inspired me 
to come up with a solution that satisfies both of those requirements: creating a domain and installing a free virtual 
load balancer to distribute traffic to the various subdomains, as well as keeping all traffic inbound to my network aimed 
at the load balancer, which then distributes any traffic that matches content filtering rules as well as passing all 
other traffic on interrupted to my home network, but <b><i>encrpyted via TLS as the load balancer communicates with other 
nodes on my network via port 443</b></i>. In this way, I have both a domain that I can build my website (and other 
services) on, I have a load balancer to distribute traffic to the appropriate on-prem resources in my home lab, and 
I've added an extra layer of security for my environment and my family's information. 

# Topology

The first thing I had to do when I was planning to implement a load balancer into my environment was to sit down and 
figure out the topology so I knew what traffic was going where. After sketching some things out, checking my reference 
videos, scratching my designs, sketching further, etc etc, this is what I came up with and implemented into my 
environment: 

![load-balancer-diagram drawio (1)](https://user-images.githubusercontent.com/73140219/186469148-aa0d5375-a6e3-4e1f-bb2b-6cc2a92e0a04.png)

As you can see, everything inbound to my private network passes through the load balancer, relying on the load balancer to 
appropriately direct the traffic. Coupled with the DNS records for my public domain to allow listed subdomains to resolve, and 
I've got public access to any of my services or resources I choose to expose to the internet, regardless of where it resides 
in my hybrid cloud infrastructure. 

# Prep Work 

The preparation for this project is not too bad, namely consisting of you <b>registering for an account with Kemp</b>, which 
will then allow you to <b>download the Kemp virtual load balancer .iso</b>. If you intend to also obtain and manage a public 
domain,  you will also need to purchase a domain name or register a free domain name from somewhere like [freenom](https://www.freenom.com/en/index.html?lang=en). 
Once you have your domain name and your Kemp .iso, you're ready to go!
