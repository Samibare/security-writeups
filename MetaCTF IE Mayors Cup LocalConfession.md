Author: Samibare

### [Local Confession](https://compete.metactf.com/611/problems#problem1675) 

During a pentest of Acme Corp, we've established an initial foothold on one of their shared Linux servers.

We know that the dev user was doing something in Python. Can you see what code they ran?

Take a look at the services on the system that are listening for network requests to find your way in.

---

**Before doing the challenge:**

Through reading the challenge details, we know that we need to identify a running Python program on the shell provided. After identifying the program, I will see what I can do from there.

**Interacting with the shell:**

Starting off with the provided link, we are brought to a website that mimics the shell. 
![LocalConfession1](/assets/LocalConfession/LocalConfession1.png)

**Scanning:**

First thing I decided to do is to run 

ps aux

To see all the running programs on the system. 

![LocalConfession2][/assets/LocalConfession/LocalConfession2.png]

Running this, we see that there appears to be a python’s built in http server on port 3000 (The default port for Python HTTP server is 8000\)

**Accessing the program:**

As this is a web server on port 3000, I can access it by utilizing curl.

curl localhost:3000

This returns the following. 

![LocalConfession3][/assets/LocalConfession/LocalConfession3.png]

Inspecting the returned service, we can see that we have the directory .python\_history available to us. By directly traversing to this directory using curl

curl localhost:3000/.python\_history

We get the following returned

![LocalConfession4][/assets/LocalConfession/LocalConfession4.png]

We can extract the flag from here,

MetaCTF{l0c4l\_c0nf3ss10ns\_0v3r\_http}

This will wrap it up for Local Confession from IE Mayors cup 2026\!

**Techniques:**

Scanning running programs: ps aux

Accessing http in the command line: curl localhost:3000

Traversing directories through request: curl localhost:3000/.python\_history

