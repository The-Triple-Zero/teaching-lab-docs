# File Manipulation Lab

In this lab, you will learn how to navigate the Linux cli, and to manipulate several files. The instructions will be provided to you. An ssh command with credentials will be presented to you in order to access the lab. Only one session is allowed per ip address.

# Instructions

1. List all of the files in your home directory. Write the command you need to do so in order to do that.
2. Write the output of listing your current working directory.
3. A file called `notes.txt` is located in the `Documents` folder. Change into that directory and print the contents of the file to the screen. Write the commands you used to do that, followed by the contents of the file.
4. Create a new directory called `my-cool-dir` under your home directory, and make an empty file called `my-new-file`. Write the commands you used to do it.
5. Find the file `delete-me` located somewhere under `/my-dir`. Write the commands you used to change into the directory holding that file.
6. Now, delete the file. Write the command you used to delete the file.
7. Find the file called `move-me` located somewhere under `/my-dir`. Write the commands you used to change into the directory holding that file.
8. Now, rename the file to `new-me`. Write the command you used to rename the file.
9. Go up one directory, an locate the file named `diary.txt`. Write how you would view a file up one directory without changing the directory.
10. From the current location, copy `/root/secrets.txt` to the current directory. Write how you would do that.
11. This diary contains a LOT of entries. However, we heard there is a funny secret in there. It was something with a pet. What was the secret?
12. Now, it's time to configure a web server. Don't worry, the template has been written for you. Paste the name you used to log into your server into your browser. You should see a crooked page there. Edit `/root/Caddyfile` to change the path from `/usr/share/caddy` to `/var/www/html`.
13. Now, copy the file over `/etc/caddy/Caddyfile`.
14. Run the command `systemctl reload caddy` once you've copied the file.
15. In your browser, paste the name of the server given to you into your browser. If things were successful, you should see a website there.