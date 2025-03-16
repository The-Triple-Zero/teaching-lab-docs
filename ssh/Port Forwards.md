# Why?

A hacker must be dynamic and must learn how to adapt to the situation he is put in. Network environments can be diverse. They can be permissive, and navigation is easy around them. In reality though, many corporate networks are complex and often restrictive, with a variety of firewalls in place between different network segments.

A key tool in the toolbelt a hacker has is `ssh`. It is pretty much guaranteed to exist on every operating system, and once a login has been found to a server, it is critical for attackers to be able to pivot across a network.

Not only as a hacker, but as a systems administrator, or developer port forwarding is quite helpful. From a "poor man's vpn" that allows you to access your home network, to reaching services inside of NATs, or taking remote resources and exposing them locally, `ssh` is a versatile tool which can do it all.

Let's walk through some common scenarios.

# Local Port Forward

## Example 1

Let's say, you compromise as machine that exposes an admin interface to `localhost:8000` only. You would be unable to reach this service under normal circumstances because you are coming from an external ip. Sure, you could execute commands from the compromised host, but most likely your toolkit will not exist there (and if it did, you should be concerned). You want to use your own tooling to gain access. How can we do that? By creating a local forward.

```bash
ssh target -L 8000:localhost:8000
```

So, what did we just do? We opened port `8000` on our local machine. This connection will be bound to the socket `localhost:8000`, which is executed on `target`. This will allow us to access the admin interface in our browser by navigating to `http://localhost:8000`.

## Example 2

For security reasons, you don't allow your home router to be configured from the public internet. You must be logged in from your home network in order to access it, but you don't have a VPN set up. However, you do have access to a jump host machine which is located on the network which can reach the router page. Let's set up a forward to the router.

```bash
ssh jump -L 8443:192.168.1.1:443
```

Now, we created an open port `8443` on our local machine bound to our router's internal ip address at `192.168.1.1` and port  `443`. In your browser, connect to `https://localhost:8443`. Now you can manage your machines from outside of the home!

## Example 3

Man, this Minecraft server I found in a pwned network would be fun to play on with my friends, but unfortunately it's not open to the public. Or, is it?

```bash
ssh pwned -L 0.0.0.0:25565:minecraft.server:25565
```

Now, we've opened up the port `25565` on our machine so that it's visible to anyone via `0.0.0.0`. This means, anyone who can directly connect to ports on our machine will be able to connect to the Minecraft server. Now, you can your friends can troll to your hearts desire (but don't steal their diamonds to make hoes, okay?)