---
title: End of Life New York Cisco Firewall Migration to Palo Alto Firewall
layout: post
post-image: "/assets/images/projects/firewall.png"
description: Firewall migration from traditional layer 4 Cisco ASA firewall cluster to the next generation Palo Alto PA-5000 series firewall cluster
tags:
- project
- network-security
- firewall-migration
- paloalto-nextgen-firewall
- cisco-firewall
- blog
---

My first role when I joined Amadeus was working as a Network Security Engineer. At that time, there was a significant project to migrate the traditional layer 4 Cisco firewalls to the next generation layer 7 Palo Alto firewalls. I brought a great deal of knowledge and experience about the Palo Alto firewalls to the Network Security services team. At that time, my team members did not have much exposure and experience to the Palo Alto Networks products and technologies. My management spotted this fact and assigned me to lead the migration project of the internet gateway Cisco firewall cluster.
<br>
<br>
![Cisco to Palo Alto Firewall Migration](/assets/images/blog/cisco_pan_firewall_migration.png "Cisco to Palo Alto Firewall Migration")
<br>
<br>
The cluster was the largest and most complicated system in our largest civil datacenter in Europe. It had multiple interfaces that interconnected multiple production network environments: Internet, VPN, GWAN, Proxy, and Core. What contributed the most to the firewall configuration complexity was the fact that the cluster was connected to the BlueCoat proxy cluster to serve the Internet traffic. As a consequence, complicated Cisco ASA old school NAT exemption ruleset was deployed to make sure that only Internet outbound traffic was redirected to the proxy cluster for the further HTTP(S) traffic filtering. Due to the system complexity and crititicality, two additional senior members were assigned to work with me. Even, one of them was holding two renowned Cisco Certified Internetwork Expert Security and Routing and Switch certifications. We quickly formed a very collaborative and high performing team and learnt a lot from each other.
<br>
<br>
![Palo Alto High Availability Design](/assets/images/blog/palo_alto_ha.png "Palo Alto High Availability Design")
<br>
<br>
As part of the project, the following main tasks were created:
- Sizing and procurement: We had to decide the Palo Alto firewall model that could cope with the large volume of traffic taking the resource intensive requirements of the next generation threat prevention and SSL decryption based on the result of the discussion with a Palo Alto Networks Major Account Senior System Engineer and worked with our procurement team to submit the procurement request.
- System design and documentation: We documented the cluster high availability design and IP address allocation for both management and data interfaces. We also worked with the Network Engineering team to identify relevant neighboring switches and allocate ports to correct VLANs.
- Hardware installation: We had to work with our datacenter hardware provision team to get the two procured firewalls installed and run cabling to connect the firewalls to the neighboring switches.
- Basic firewall configurations: We configured basic configurations likes the management interface IPs, NTP, the Palo Alto update service and data interface IPs, etc. At the same time, we reserved the allocated IPs in our IPAM system. To avoid mistakes that could brought the systems to live, we had to shutdown all the data interfaces during the configuration phase.
- Advanced firewall configurations:
    - At that time, the firewall migration toolset was still in its early phase and used py the Palo Alto professional services exclusively. Therefore, we relied on them to convert the Cisco firewall ruleset to the Palo Alto security policies. We still had to do due diligence on checking the accuracy of the conversion.
    - We worked together to manually convert the complex NAT rules and routes to the Palo Alto counterparts. There were some historical rules and routes none of us knew why they were there. We even have to look up the historical change records and search relevant logs on our logging platform in an effort to understand them and figure out if they needed to be cleaned up.
    - We set temporary IPs for the data interfaces, brought them up and run ping tests to make sure that there were layer 3 connectivities between the new cluster and the rest. This was done to not only test the migrated routes but also make sure that the switch configurations implemented by the Network Engineering team was trustworthy.
- Cutover: After all the configurations and due diligence checks were done, we scheduled a change request and got the change management approval for the cutover. During the cutover, we shut down the interfaces on the Cisco firewalls, brought up the data interfaces on the Palo Alto cluster and sent gratuitous ARP requests from it to update the APR caches of the neighboring network devices. At the same time, we monitored logs generated from the Palo Alto firewalls on our logging platform to make sure that all the routing, security and NAT policies were working as expected.
<br>
<br>
The end results were very impressive. The cutover went seamlessly without any incidents and was successful at the first attempt. Our special efforts won us the STAR (Special Thanks And Recognition Award) award.
