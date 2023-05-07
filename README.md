Download Link: https://assignmentchef.com/product/solved-iv1301-firewall
<br>
The learning objective of this lab is for students to gain the insights on how firewalls work by playing with firewall software and implement a simplified packet filtering firewall. Firewalls have several types, and the two most commonly known are the packet filter and application level gateway (application firewall). Packet filters act by inspecting the packets; if a packet matches the packet filter’s set of rules, the packet filter will either drop the packet or forward it, depending on what the rules say. Packet filters are usually stateless; they filter each packet based only on the information contained in that packet, without paying attention to whether a packet is part of an existing stream of traffic. Packet filters often use a combination of the packet’s source and destination address, its protocol, and, for TCP and UDP traffic, port numbers. In this lab, the focus is on packet filtering firewalls.

Lab environment

Network setup

To conduct this lab, you need four machines: a firewall, an inside host, an outside host, and an attacker. You will use an LXC container for each of these machines. Let the outside network be 10.0.10.0/24, and the inside network be 10.0.20.0/24.

Topology

Tools

wireshark – Sniffer and protocol analyzer tcpdump – Command-line based sniffer

netwox – Tools to generate packets and spoof network traffic netcat (nc) – Lots of different tools, can be used for a simple client/server. iptables – tool to configure the tables provided by the Linux kernel firewall (Netfilter modules) ufw – a firewall configuration tool for iptables (ufw = <strong>u</strong>ncomplicated <strong>f</strong>ire<strong>w</strong>all)

Tasks

Connecting to the hosts




Open one terminal window and run the following commands to start the LXC containers:




lxc start attacker  lxc start inside-host lxc start outside-host lxc start firewall




Now, connect to these containers. You will open new terminal window for each container, and run each command in separate window:




lxc exec attacker /bin/bash lxc exec inside-host /bin/bash lxc exec outside-host /bin/bash lxc exec firewall /bin/bash




Once you run the command, you will get into the root terminal in the container.




Lastly, test connectivity by pinging each host from all others.




<h2>Task 1 – Building a Firewall</h2>

In this task you will use ufw to build a simple firewall for your network.




1.1 Default Policy




It is good practice to take a conservative approach, and state that everything that is not explicitly allowed, is forbidden:




<ul>

 <li>Set the default policy for the ufw firewall to drop (deny) all packets.</li>

</ul>




1.2 Network Permissions




Now it is time to start allowing some carefully chosen traffic through the firewall.




<ul>

 <li>Create a rule that allows all traffic originating from the internal network arriving to the internal interface of the firewall host.</li>

 <li>Create a rule that allows all traffic originating from the firewall to reach the internal network.</li>

</ul>




Make sure you now can reach the internal network from the firewall, and the firewall from the internal network.




<strong>Question 1</strong>: What rules do you need in order to set up the default policy, and to allow communication with the firewall and the internal network?




<strong>Question 2</strong>: with ufw you can choose between “deny” and “reject” when you want to disable traffic. What is the difference between deny and reject? What are the advantages/disadvantages? Make an experiment where you set up the firewall to reject instead of deny, and explain what you see.




Report your ufw configuration for this part by using the command ufw status verbose.




1.3 Permitting a Service




SSH (Secure shell) is a common service for remote management of firewalls. Create rules which allow a host from the external network to connect to the firewall with SSH. SSH runs on port 22 and uses only TCP.

Make sure that the SSH daemon sshd is running on all hosts by executing service ssh restart.




<h3>Note</h3>

To access any of the hosts with ssh, you must use the following user account:

username: ubuntu password: ubuntu

e.g., ssh <a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="67120512091312271f491f491f491f">[email protected]</a>







Verify that

<ul>

 <li>you <strong>can</strong> connect to the firewall with ssh from the other hosts,</li>

 <li>you <strong>cannot</strong> connect directly from your external host to the internal host with ssh, and</li>

 <li>you <strong>can</strong> connect from your external host to the internal host if you first connect to the firewall with ssh and then connect from there with ssh to the internal host (note: think about which username you should use when logging in to the hosts with ssh).</li>

</ul>




<strong>Question 3</strong>: What are the advantages and disadvantages with opening up for ssh login from the outside?




Report your ufw configuration for this part by using the command ufw status verbose.




1.4 Stateful Filtering




In most cases we want to allow hosts on the internal network to connect to the external network, e. g., the Internet. However, we do not want hosts on the Internet to be able to connect to hosts on the internal network.




<ul>

 <li>Create a rule (or rules) to allow the inside host to establish connections to the outside. Keep blocking (dropping) all other connection attempts from the outside.</li>

</ul>




Verify that

<ul>

 <li>you <strong>can</strong> connect directly to the outside from the inside, you <strong>cannot</strong> connect directly from the outside to the inside.</li>

</ul>




<strong>Question 4</strong>: How can you verify that traffic from the outside going in is prevented, while traffic from the inside going out is allowed? Consider TCP, UDP as well as ICMP traffic. What tests do you perform to validate your configuration?




Report your ufw configuration for this part by using the command ufw status verbose.




1.5 Opening Ports




Sometimes, you want computers on the outside of the network to have access to a specific service on the inside. Your task now is therefore to add rules to your firewall, so that the outside host can reach a service at port 9000 on the inside host, for UDP as well as TCP. Use netcat (nc) for your service, both on the client and the server side.




<strong>Question 5</strong>: How can you verify that traffic to the service on the inside host at port 9000 is allowed, but no other services?




Report your ufw configuration for this part by using the command ufw status verbose.




1.6 Blocking Ports




Sometimes you do not want your internal users to be able to connect to the Internet on a specific port at all. One commonly blocked port is 135 which is used for Windows file sharing. Make sure your firewall blocks all traffic on port 135 (both TCP and UDP) from the computers on the internal network.




<strong>Note: </strong>

Actually there are more ports involved (135-139 on older Windows machines and 445 on more recent ones) but in this lab it is enough to block 135 as one example.




<strong>Question 6</strong>: How can you verify that the inside host cannot reach port 135 on the outside, but no other outgoing traffic is blocked?




Report your ufw configuration for this part by using the command ufw status verbose.




1.7 Verifying Your Setup




Verify your setup in a systematic manner, making sure all rules work correctly. To test a certain rule, you can listen to a port with netcat and try to connect to it using netcat or telnet:




On the receiving host run: nc -l PORT

On the sending host run: telnet IPADDRESS PORT or nc IPADDRESS PORT




After completing the tasks above, you should have the following rules active:

<ul>

 <li>Connections on <strong>port 135</strong> from the inside hosts are blocked.</li>

 <li>Connections on the <strong>SSH port</strong> are allowed to the firewall host from the outside.</li>

 <li>Connections on the <strong>port 9000</strong> are allowed to the internal hosts from the outside.</li>

 <li>Connections directly <strong>to and from the firewall</strong> are allowed from the internal networks.</li>

 <li>Connections <strong>from the inside</strong> are allowed out.</li>

 <li>Connections <strong>from the outside</strong> are blocked.</li>

</ul>

<h2>Task 2 – Defending Against SSH Brute-force Attacks</h2>




You have allowed connections to the SSH port (hence, its service) of the firewall from any host on the outside network. There is nothing that would avoid a brute-force attack or a Distributed Denial of Service (DDoS) attack because there are no limitations on the amount of times that one particular host can try a username and a password (besides the ones that the SSH server implementation might have internally).




Examine how you can use ufw to prevent excessive load on the firewall from SSH attacks. Verify that it works.




<strong>Question 7</strong>: How does ufw support protection against denial of service attacks? Explain how you configure your firewall.




<strong>Question 8</strong>: How can you verify that the protection works? Explain the experiments you perform.




Report your ufw configuration for this part by using the command ufw status verbose.

<h2>Task 3 – Ping and the Internet Control Message Protocol (ICMP)</h2>

The ping tool is usually used to test the reachability of a host that implements the Internet protocol (IP). It operates by sending packets using the Internet Control Message Protocol (ICMP) of the type ‘echo request’ to the host whose reachability is to be tested, and processing the response from that host (if any).




Ping can also be used in attacks, such as ping flooding, ping of death, and smurf attacks. Your overall task here is to design firewall rules that will allow inside-host to ping outsidehost, but at the same time prevent an attacker from doing an attack where large amounts of ICMP echo requests or ICMP echo replies reach the inside-host.




There are no ufw commands to directly set up rules regarding ICMP echo requests and replies, so you may have to work with iptables to create such rule sets.




<h3>Blocking ICMP Echo Requests</h3>




ping the inside-host from the outside-host and describe what happens.

Set up a rule on your firewall to block all ICMP echo requests from the external network to your internal network. ping the inside-host again and describe what happens.




<h3>Rejecting ICMP Echo Requests</h3>




Replace the block rule above with a new rule which rejects all ICMP echo requests from the external network to your internal network.




Verify the following:

<ul>

 <li>That you can ping from the inside-host to the outside-host.</li>

 <li>That the outside-host cannot ping the inside-host.</li>

 <li>The firewall can ping both hosts.</li>

</ul>




<strong>Question 9</strong>: Can you ping from the outside-host to the inside interface of the firewall? Why/why not? Can this have any security implications?

<strong> </strong>

<h3>Blocking ICMP Echo Replies</h3>

<strong> </strong>

Let the attacker ping the outside-host while spoofing the source address by using the insidehost’s address as source address. Use netwox if necessary.




Create a rule to prevent the ICMP Echo Replies from reaching the inside-host in the scenario above.




In your report, specify the rules you now have in your firewall. You can use the following commands to examine the rules:

<ul>

 <li>ufw status numbered</li>

 <li>ufw status verbose iptables –list –verbose</li>

</ul>

<strong> </strong>

<h3>Logging and Limits</h3>




One important task for a firewall is to log rejected or dropped packets, making it easier to trace attacks. A rule can be created with the jump target NFLOG to save information to a system log.




Create a rule that logs all rejected ping messages. Make sure the string “Ping rejected by &lt;your name&gt;:” is written in the log message. Check the system log so the traffic is saved. This can be done with:

– tail -f /var/log/ulog/syslogemu.log

The module limit can be used to limit how often a rule can be triggered. Use the limit module to make sure no more than 5 pings each minute are saved to the system log.




In your firewall there is now no possibility for the inside-host to ping the outside-host. Your final step in this task is now to configure the firewall so that you prevent the attacker from successfully creating large amounts of fake ICMP echo replies that reach your inside-host, while still making it possible to ping the outside-host from the inside-host.




The key here is to allow occasional ICMP replies from the outside to pass the firewall to the inside, but to make sure that ICMP replies arriving at a high rate cannot reach the inside. It is up to you to define high rate versus occasional ICMP replies.




<strong>Question 10</strong>: Explain how you can limit the amount of ICMP echo replies that reach the inside host. How can you verify that it works?




Your report must include a specification of the rule set created for this task. You should also provide packet traces (using e.g., wireshark) to demonstrate the behavior of your firewall.