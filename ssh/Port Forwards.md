# Why?

A hacker must be dynamic and must learn how to adapt to the situation he is put in. Network environments can be diverse. They can be permissive, and navigation is easy around them. In reality though, many corporate networks are complex and often restrictive, with a variety of firewalls in place between different network segments.

A key tool in the toolbelt a hacker has is `ssh`. It is pretty much guaranteed to exist on every operating system, and once a login has been found to a server, it is critical for attackers to be able to pivot across a network.

Not only as a hacker, but as a systems administrator, or developer port forwarding is quite helpful. From a "poor man's vpn" that allows you to access your home network, to reaching services inside of NATs, or taking remote resources and exposing them locally, `ssh` is a versatile tool which can do it all.

Let's walk through some common scenarios.

# Local Port Forward

With a local port forward, you're concerned with making remote content available locally. You will see a few examples down below.

## Scenario 1

Let's say, you compromise as machine that exposes an admin interface to `localhost:8000` only. You would be unable to reach this service under normal circumstances because you are coming from an external ip. Sure, you could execute commands from the compromised host, but most likely your toolkit will not exist there (and if it did, you should be concerned). You want to use your own tooling to gain access. How can we do that? By creating a local forward.

```bash
ssh target -L 8000:localhost:8000
```

So, what did we just do? We opened port `8000` on our local machine. This connection will be bound to the socket `localhost:8000`, which is executed on `target`. This will allow us to access the admin interface in our browser by navigating to `http://localhost:8000`.

## Scenario 2

For security reasons, you don't allow your home router to be configured from the public internet. You must be logged in from your home network in order to access it, but you don't have a VPN set up. However, you do have access to a jump host machine which is located on the network which can reach the router page. Let's set up a forward to the router.

```bash
ssh jump -L 8443:192.168.1.1:443
```

Now, we created an open port `8443` on our local machine bound to our router's internal ip address at `192.168.1.1` and port  `443`. In your browser, connect to `https://localhost:8443`. Now you can manage your machines from outside of the home!

## Scenario 3

Man, this Minecraft server I found in a pwned network would be fun to play on with my friends, but unfortunately it's not open to the public. Or, is it?

```bash
ssh pwned -L 0.0.0.0:25565:minecraft.server:25565
```

Now, we've opened up the port `25565` on our machine so that it's visible to anyone via `0.0.0.0`. This means, anyone who can directly connect to ports on our machine will be able to connect to the Minecraft server. Now, you can your friends can troll to your hearts desire (but don't steal their diamonds to make hoes, okay?)

# Remote Port Forward

Remote port forwards allow you to take local content and make it available on the remote.

## Scenario 1

You've about to perform a physical penetration test on a customer site. You know they have a pretty locked down external environment, but the word is they think that their firewall means they don't need any internal security controls. I mean, they can't get in, so how would they even hack us? You know that they only allow encrypted web traffic to the external internet. Maybe they don't have a firewall with deep packet inspection, and are just checking the port number?

In preparation, you acquire a public cloud instance and configure your `ssh` port to port `443`. Next, you create a Raspberry pi which when powered on will run a script with the following line:
```bash
ssh my_vps -R 2222:localhost:22
```

You perform the penetration test and successfully plant the device on the customer network. Now, you head back home and log into your VPS to find port `2222` now has a connection. It sends you to port `22` on the Raspberry Pi you set up which is inside the customer network. Now, you can access all of their internal machines!

## Scenario 2

After you had your fun trolling in the hacked Minecraft server, you've reformed your ways and decide to set one up yourself. However, you don't want to expose the IP address of your house where you run it. Thankfully, there is a low-latency VPS instance you can connect to.

You grab your terminal, and run the following command:
```bash
ssh my_vps -R 0.0.0.0:25565:192.168.1.5:25565
```

You just opened up port `25565` to the world to access on your VPS instance, and it will be routed over to your local Minecraft server running at `192.168.1.5:25565`. Now, nobody will find out your IP, and you get to play with friends!

# Dynamic Forwards

In the previous sections, we saw that though there is much flexibility in creating tunnels with both directions. However, we were limited to one socket pair. What if we could bypass that and open all all ports to go through? Actually, you can! You can have `ssh` set up a SOCKS5 proxy for you. But... with a caveat. It will only work for TCP traffic. It is not possible to forward UDP ports with the built in `ssh` SOCKS5 proxy.

This ability is incredibly useful to bypass firewalls and pivot across a network. It's also commonly known as the "poor man's vpn" because no additional configuration is necessary except for `ssh` access to a server on the network you'd like to connect to things.

## Scenario 1

Let's look back to Scenario 2 from the local forwarding section. It's great that we can now connect to our router, but what about other services on our network such as our Minecraft server?

Let's alter the command to extend its functionality:
```bash
ssh jump -D 1080 
```

What did we just do? We opened a SOCKS5 proxy on our client machine's port `1080`. Now, any SOCKS5 compatible client can forward traffic through our `jump` server to reach internal services. For a browser, this is usually done in the settings, or by using an extension like FoxyProxy. Many applications also allow for SOCKS5 configuration.

However, our Minecraft client doesn't allow for SOCKS in this specific case. How can we overcome it?
```bash
mkfifo /tmp/f; cat /tmp/f | nc -klvp 25565 | nc -x localHost:1080 -X 5 192.168.1.5 25565 > /tmp/f; rm /tmp/f
```

Wait, what is that? That uses netcat to create a listening port on `25565` and pipes the output to the input of a netcat connection using the SOCKS5 proxy we created earlier to connect to out Minecraft server. Now, the output of that connection is redirected to a pipe which is being read and put into the input of the listening port.

In short, you simply opened a new port `25565` on your local host to connect to your Minecraft server.

Or, we can do it in a simpler manner using `socat`.
```bash
socat TCP-LISTEN:25565,bind,reuseaddr,fork SOCKS5-CONNECT:localhost:1080:192.168.1.5:25565
```

## Scenario 2

In the previous section we mentioned a physical penetration test. Though it's great we can execute commands from our Raspberry Pi, wouldn't it be more useful to route arbitrary IP addresses over it from our local machine? With a few commands you can!

```bash
ssh localhost -p 2222 -J my_vps -D 1080
```

Here, we jump through the VPS to connect to `localhost:2222`. Now, we create an open port on `1080` to route through the raspberry pi.

Once again, we can route any arbitrary traffic we would like over this connection just as in the previous scenario.

# VPNs

Who needs a poor man's vpn, when we can actually create a real one? However, keep in mind this has some limitations compared to dynamic forwards.

- It is a non-standard setting, so it is not set by default
- It will require root access on the remote host, as well as the client
- Though it will allow all Layer 2 or 3 traffic to go across the network without a special configuration like a Dynamic Forward, it is generally slower than a Dynamic Forward due to TCP / TCP encapsulation as opposed to the remote making the TCP request on your behalf with a SOCKS proxy
- When using a Layer 2 tunnel, you're doing ARP over a VPN over TCP, which will be very slow

In many situations, this is not possible. However, it opens up the ability for full forwarding of Layer 3 traffic in TUN mode. This includes ICMP (ping), TCP, and UDP. Or, you can even forward Layer 2 traffic in TAP mode. The performance is likely to be terrible. However, when it's needed in a pinch, man is this helpful!

To permit both `point-to-point` (layer 3) and `ethernet` (layer 2) tunnels, you must set the following config on the ssh server you intend to use this feature with.
```
PermitTunnel yes
```

## Layer 3 tunnel

Layer 3 tunnels allow you to encapsulate packets from the IP layer and above (UDP, and TCP). This will allow you to send pings over the `ssh` connection.

### Scenario 1

You happen to be at a train-station with free, open WiFi like I am while writing this. You find out all VPNs you have tried are blocked. You want to protect your traffic from your ISP (the train station), and would rather send the traffic back home first before going out to the internet.

Establish a tunnel. From `root` on your machine, you ssh into `root` on the remote host
```bash
sudo ssh root@remote -w any
```

This will establish a layer 3 tunnel using any tun device available on both machines. You should now see a device like this created on both machines:
```
4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 500
    link/none 
```

On both sides, set the tunnel device up
```bash
sudo ip l set tun0 up
```

On the remote side, add an ip address, and set up NAT out of this interface. The assumption is you will use iptables.
```bash
sudo ip a add 192.168.150.1/29
# Ensure forwarding is enabled
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo iptables -P FORWARD ACCEPT # This is potentially dangerous depening on what this machine is used for
sudo iptables -t nat -A POSTROUTING ! -o tun0 -j MASQUERADE
```

On the local side, add another ip address in the same range, and set it as the default route.
```bash
sudo ip a add 192.168.150.2/29
sudo ip r add default via 192.168.150.1
```

From here, you should have all traffic going over the tunnel, and can keep you traffic safe from those pesky snoopers at the train station.

## Scenario 2

You have a local network of secure virtual machines on `172.18.0.0/16` which must route all traffic to a secure physical site. Unfortunately, the VPN is down. You need to do some critical, time-sensitive work. To make it even more difficult, policy states all host traffic must go through the company proxy. Therefore, you can't route all traffic through the VPN. Can you come up with a work around? Of course you can!

On your local machine, you pre-allocate a tun device to use for the ssh connection and give permission for your user to modify it. Then, you set it up and give it a unique ip address.
```bash
ip tuntap add name tun0 mode tun user myuser
ip l set dev tun0 up
ip a add 192.168.111.2/30 dev tun0
```

Now on the remote, you do the same with the correct user, and another ip address.
```bash
ssh remote -- 'ip tuntap add name tun0 mode tun user myremoteuser; ip l set dev tun0 up; ip a add 192.168.111.1/30 dev tun0'
```

