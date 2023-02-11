# TryHackMe-Daily-Bugle-Walkthrough-2-ways

## Enumeration
Starting out with a basic scan we can see there are a couple of ports open.
![nmap](https://user-images.githubusercontent.com/103790652/218285114-f4b6987c-977e-4069-a4fe-c0d36757781e.png)

The webpage served is pretty basic but a gobuster scan reveals more about the site (notable pages: admin + multiple others that suggest possible file upload).

![gobuster](https://user-images.githubusercontent.com/103790652/218285545-78f84b84-78b8-4681-9bf4-2fe41f5f99fe.png)


Running a quick vlunrability scan against the site returns an interesting result - the site uses Joomla! CMS and is vulnerable to SQLi!
![vuln result](https://user-images.githubusercontent.com/103790652/218285261-cada9046-c786-4a7c-932f-7328a8112efc.png)

## Exploitation

Running the exploit against the page yields a few usernames, as well as a bcrypt hash.
![joom](https://user-images.githubusercontent.com/103790652/218285303-8f4b9622-a418-4c18-a0aa-91c69d3053f0.png)

Feeding the hash to JohnTheRipper yeils the great password of spiderman123 after a wait.

Tried to log in via ssh using the obtained usernames + password but had no luck so returned to the webpage scan results.

Found a log in page at /administrator and tried the credentials again -- success.

Upon log in, greeted by generic Joomala dashboard.

Tried to upload shell file via the media folder but there seemed to be some sort of client side filtering prventing the file being delivered/ server side hiding of any uploaded files so I decided to check elsewhere on the site for easier access and return later.

After some searching, found that files could be added/written within the website template folder and successfully uploaded a reverse php shell.
![shell](https://user-images.githubusercontent.com/103790652/218285526-d0671de3-2257-46bd-928c-f1851e9a6602.png)

Navigating to the shell's URL triggers it and a local listener is able to catch the response - we're in.
![path](https://user-images.githubusercontent.com/103790652/218285613-44560eff-7cdc-41d4-9292-4b8ebdd587e4.png)
![catch](https://user-images.githubusercontent.com/103790652/218285615-231dce4d-4fbe-4f0e-93a5-7400bb196362.png)

Achieved a stable shell via experimenting with the "magic" route found here as I hadn't used it before:
https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/

Now moved to /tmp as Read/Write permissions were present and uploaded linpeas.sh (https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) via local machine web server.

Linpeas showed 2 possible routes of privilege escalation: 

A clear text password:
![pass](https://user-images.githubusercontent.com/103790652/218285845-5e3ee25c-65e9-4112-a154-f8ecae9abb24.png)

and direct root access via CVE-2021-4034:

![cve](https://user-images.githubusercontent.com/103790652/218285862-1969f37d-9dde-46eb-9605-bc7909a72aeb.png)

## Password / account privesc

Very simply, this just involved switching to the user jjameson (found in /home) with the password found by linpeas. 
![switchuser](https://user-images.githubusercontent.com/103790652/218285981-b737f694-059e-4d87-ad43-3f57345e66fe.png)
Once the account was switched, the user.txt flag was readable from the Desktop.

Root privesc in this path was done first by finding out what I could get away with (run):
![sudo -l](https://user-images.githubusercontent.com/103790652/218285995-45da6dd1-5498-4726-98ca-e7612ded7f16.png)

Checking GTFObins it was apparent that yum can spawn interactive root shell by loading a custom plugin when it doesn't drop sudo rights
![gtfobins](https://user-images.githubusercontent.com/103790652/218286023-b2ea3946-d28d-441c-8360-567673fb6f2a.png)









