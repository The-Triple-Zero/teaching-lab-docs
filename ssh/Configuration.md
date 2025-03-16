# What is an SSH config?

As we covered in [Getting Started](/ssh/getting-started), `ssh` is a very powerful tool. However, it can very quickly become overwhelming to remember all of the flags and ip addresses to specify when connecting to a particular machine. Wouldn't it be easier if we could just do something like this?

```bash
ssh kali
```

Well, it turns out that you can! You just need to provide `ssh` with the correct configuration settings, and it will understand what you need.

You can do this by adding the following into the file `~/.ssh/config`:
```ssh
Host kali
	Hostname <my_server_ip>
	User <my_username>
	Port 22
	IdentityFile ~/.ssh/id_ed25519
```

Now, anytime you specify `ssh kali`, it is as if you're doing the following:
```bash
ssh <my_username>@<my_server_ip> -p 22 -i ~/.ssh/id_ed25519
```

In summary, it's a file to allow us to specify the settings we'd like the command line to run, without needing to type the every time.

# Some useful settings

Configuration files are incredibly powerful. You have the ability to use wildcards, and to match various parts of a configuration to specific types of configs. Additonally, you can specify global options that will get used in all hosts, unless otherwise overridden by a more specific config.

The following is a template that has developed / collected over time and is a pretty solid start to utilize the full potential of an `ssh` config.

```
# Allow custom configurations
Include ~/.ssh/config.d/*

# Open a SOCKS5 proxy I can route all traffic over in my home network
Host jump.home
  Hostname <public_ip>
  Port <external_port>
  User <my_user>
  DynamicForward 1080
  IdentityFile ~/.ssh/home-infra

# Configure github so the correct settings are understood
Host git github.com
  Hostname github.com
  Port 22
  User git
  IdentityFile ~/.ssh/github

# Good for an ephemeral environment where ssh host keys change often
Host *.test.net
  User ubuntu
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null

# Set a default user for all home infrastructure.
# Disable the Control Master because we assume
# we're on a LAN with low latency
# Add a different ssh key for a different purpose
Host 192.168.*.*
  User ubuntu
  ControlMaster no
  IdentityFile ~/.ssh/home-infra

# Allow resolution of `.onion` domains over the standard TOR socket.
Host *.onion
  VerifyHostKeyDNS no
  ProxyCommand nc -x localHost:9050 -X 5 %h %p

# Set up default settings
Host *
  ControlMaster auto
  ControlPath ~/.ssh/socket/%r@%h:%p
  ControlPersist 10m
  ConnectionAttempts 10
  ConnectTimeout 10
  IdentitiesOnly yes
```

We will break this down some in the following sections.

## Global settings

There is an order of precedence when specifying a config. The top-most parts of the file are evaluated first, and anything which follows will be ignored. This means, in order to declare global values that you generally want unless specified otherwise, you must stick them at the bottom.

You see this in the following section:
```
Host *
  ControlMaster auto
  ControlPath ~/.ssh/socket/%r@%h:%p
  ControlPersist 10m
  ConnectionAttempts 10
  ConnectTimeout 10
  IdentitiesOnly yes
```

## Wildcards

As you can see, there are a few incomplete hostnames above. The wildcard allows the interpreter to fill in the blanks with what is to come. This allows us to describe general configurations that match a subset of hosts.

Here is one example:
```
Host 192.168.*.*
  User ubuntu
  ControlMaster no
  IdentityFile ~/.ssh/home-infra
```

This matches everything that contains `192.168.` followed by anything of any length, followed by a `.`, followed by anything of any length. The assumption is the user will put in a valid ip address, but technically it matches more than that.

## Multiple host values

Not to be confused with `hostnames`, the host field allows you to specify numerous hosts. Therefore, if any is seen, it will evaluate to the settings given below. Just as you see in this example:
```
# Configure github so the correct settings are understoof
Host git github.com
  Hostname github.com
  Port 22
  User git
  IdentityFile ~/.ssh/github

```

## Includes

Includes allow you to take configurations from more than one ssh config file. This is helpful when splitting settings up into specific projects. It can become easier to manage single well-defined files per project / use case than a single `~/.ssh/config` that's hundreds or even thousands of lines long. For example, this includes all files in the `~/.ssh/config.d` directory.
```
Include ~/.ssh/config.d/*
```

## Forwarding

This allows a user to connect a local port to a remote port through the specified host, to open a port on the remote to map back to the local machine, or to allow arbitary port forwarding through the host via a SOCKS5 proxy. This is further explained in [Port Forwards](/ssh/port-forwards).

```
Host jump.home
  ...
  DynamicForward 1080
```