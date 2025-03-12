#  What is ssh?

SSH stands for **S**ecure **SH**ell. It was the solution created to replace rsh (Remote Shell), the tool administrators used to manage remote machines. At the time, rsh lacked encryption and was therefore prone to hackers sniffing the network traffic to gain password access

A bit of history:
- Tatu Ylönen, a Finnish university researcher witnessed a password sniffing incident on his university network
- Ylönen decided to learn about cryptography, spoke with some friends, and decided to write SSH as an open source software
- It became an instant hit
- Ylönen developed a company, then started to limit the licensing. As a result, forks developed including the OpenSSH fork, which is an industry standard tool today

# Why do we use it?

It has become the defacto tool used to administrate UNIX, and its derivatives commonly known as \*NIX style machines, such as Linux, and BSD. It is an absolutely essential tool to programmers, systems administrators, and *especially* hackers. These days, OpenSSH offers more than just access to a commandline. It offers advanced networking capabilities which get used in firewall circumvention, the ability to reach specific remote desktop applications from your terminal and can provide an encrypted and compressed tunnel which can be used to send application data over it.

# How do we use it?

Most people don't get past the absolute basics and it's truly a shame to see such a valuable swiss-army knife to be so underutilized. Here are the bare essentials to log into a machine.

```bash
ssh [user@]<server> [-p <port>] [-i <identity_file>]
```

Let's break this syntax down bit-by-bit:
- `ssh` - This is the `command` that you supply to your shell in the terminal. Everything that follows it is known as an `argument`. Anything which starts with a `-` is known as a `flag`.
- `[user@]` - The square braces denote this is an optional parameter. This indicates you can specify a `user` at (`@`) the server. For example: `ssh stew3254@my-server`. If you don't specify a user, the default is your current username.
- `<server>` - The angle brackets denote this is a required parameter. You *must* specify a server to connect to. For example: `ssh my-server`.
- `[-p <port>]` - The `-p` is an optional flag specified by a required field called `port`. If you use this flag, you can specify which port you want ssh to connect to. If it isn't provided, the default value is port `22`.
- `[-i <identity_file>]` - The `-i` is an optional flag specified by a required field called `identity_file`. If you use this flag, you can specify the path to a file containing your public key you want to connect to the server with. If it isn't provided, the default value is port to not use a key.

To ssh into your Kali Linux vm, you might write something like this:
```bash
ssh kali@192.168.200.254 -i ~/.ssh/id_ed25519
```


# How can I learn more?

To learn more about using ssh, you can read the manual page for more information. At the bottom, it will tell you the names of other related manual pages.

```bash
man ssh
```

There are also numerous guides online which will walk you through many of the basics of ssh, but the most detailed information generally comes from the source.