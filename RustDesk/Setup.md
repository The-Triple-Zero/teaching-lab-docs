# Setting up RustDesk

RustDesk is a fast, open-source remote access software that is designed to replace legacy protocols like RDP (Remote Desktop Protocol), and VNC (Virtual Network Computing). It provides a clean, and simple interface with easy control for things like USB redirection, audio redirection, and port-forwarding. They offer native clients for every major operating system, and also a [web-based client](https://rustdesk.com/web/) for convenience. Do note, native clients result in faster connections, and likely have more supported features than the web client.

## Installation

In order to use RustDesk, you will need to install the native client at least on the system you intend to connect to.

To get started with the RustDesk client, check out the [downloads page](https://rustdesk.com/download). This should redirect you to the latest release on GitHub. From there, you can download the appropriate version for your operating system.

To install RustDesk on a Debian-based operating system, download the "Ubuntu" `.deb` file. Almost certainly you want the x86-64 architecture.

Here is how you can do that via the commandline.
```bash
wget -O /tmp/rustdesk.deb https://github.com/rustdesk/rustdesk/releases/download/1.3.8/rustdesk-1.3.8-x86_64.deb
```

From here, `apt-get install` the package.
```bash
sudo apt-get install -y /tmp/rustdesk.deb
```

There is nothing further you need to do from here

## Configuration

There are two different configurations available. On the machine you intend to control as a remote host, or as a client to connect to the remote host.

### As a remote host to connect to

For the purposes of this lab, the following settings have been selected for you, and are available to put in the configuration file located at `~/.config/rustdesk/RustDesk2.toml`. You must place them under the `[options]` section of the file.

```toml
allow-linux-headless = 'Y'  
verification-method = 'use-permanent-password'  
approve-mode = 'password'  
direct-server = 'Y'  
allow-remote-config-modification = 'Y'
```

To do this via the commandline, you can paste the following. If you are not on Kali Linux, or not running as the `kali` user, please be sure to update the path to reflect the accurate home directory before pasting. Be sure to only execute this command once.
```bash
cat <<EOF >> /home/kali/.config/rustdesk/RustDesk2.toml
allow-linux-headless = 'Y'  
verification-method = 'use-permanent-password'  
approve-mode = 'password'  
direct-server = 'Y'  
allow-remote-config-modification = 'Y'
EOF
```

Now, you can set a permanent password to use. When you type in your password, it will not show anything on the screen for privacy reasons. It is still recording your input.
```bash
read -rs password
sudo rustdesk --password "$password"
```

Now, restart the service to ensure the changes took affect.
```bash
sudo systemctl restart rustdesk
```

For good measure, ensure it runs on startup as well if you'd like to be able to connect every time the machine is started.
```bash
sudo systemctl enable rustdesk
```

Now, you can get the ID of the client.
```bash
sudo rustdesk --get-id
```

You should see a 9 digit number like the following:
```
123456789
```

Use this number in your client to connect to your machine, and provide the password you supplied earlier. You should be all set from the remote host side.

### As a client

You can feel free to modify the settings as you like to meet your needs. However, adaptive scaling is recommended so that if the screen is able to adjust its resolution, it will adapt to your screen size.

From here, RustDesk should be available for use immediately after setup. No additional configuration must be done. However, at this time there are improvements that can be made to further enhance user experience.

## Additional Configuration

Currently, with the default settings you will receive a screen with a resolution size of 1240x780. This is tolerable in a web browser, but will result in a loss of clarity.

In order to increase the resolution size, you can create a fake monitor for Xorg to display to. **Beware**, if you do this, the graphical console will no longer work from the UI. This will be the only way to connect to the machine. However, it will allow you to customize the resolution size to your liking.

In order to do that, navigate to the directory `/usr/share/X11/xorg.conf.d/`. You will find a file called `99_rustdesk.conf.disabled`. Rename that file to remove the `.disabled`. This will give you a new screen with a `1920x1080` resolution by default, but these values can be customized. However, customization of this is out of the scope of this documentation.

```bash
sudo mv 99_rustdesk.conf.disabled 99_rustdesk.conf
```

Optionally, you can do it the shorthand way for less typing.
```bash
# Or, the shorthand way
sudo mv 99_rustdesk.conf{.disabled,}
```

Restart your machine for the changes to take place.
```bash
sudo reboot
```