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
- `[user@]`
- `<server>`
- `[-p <port>]`
- `[-i <identity_file>]`