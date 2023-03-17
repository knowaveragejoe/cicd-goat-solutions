# Hearts

There is no code repository involved in this challenge.

Instead, we are given a clue about a Jenkins user that is capable of configuring Jenkins agents.

First, we have enough permissions as the Jenkins user `alice` to show other users. This shows us a few other users, the most interestinf of which is `knave`. 

Doing a little research on hacktricks, we find that Jenkins has no throttling on login attempts for users(still?), so we can try bruteforcing this user's password with the rockyou password list.

I found a script that attempts password spraying against Jenkins using a provided wordlist, and it was able to recover the password for `knave` in a few minutes:

https://github.com/gquere/pwn_jenkins/blob/master/password_spraying/jenkins_password_spraying.py

With that in hand, we can log in as this user and configure additional build agents. Build agents must be configured with method of connecting to them(how else does Jenkins know how to interact with them?), so this gives us a potential avenue of attack.

My intiial thought was to use the prefix or suffix connection command fields under advanced options connecting to the agent via SSH, but this was ultimately futile because the password is IN the Jenkins credential store, not the environment around it(where those prefix/suffix commands would run).

Next, I tried using netcat to listen for incoming connections on port 22 and see if I could glean anything that way. While the agent did connect, it only sent over the initial string of the library that was performing the connection. Turns out it needs a bit more of a handshake to cough up anything useful for us.

I did turn to the challenge hints to get this far, which pointed me at a tool for MITM'ing SSH connections. It looked like a lot of work to set up, so in my research for alternatives I came across a much simpler tool: https://github.com/ssh-mitm/ssh-mitm

While I was able to get the agent to connect to this tool, it took a while before I was able to figure out that I needed to enable interactive login for the incoming connection. I didn't realize at first that the secret we're after is actually the password for a username/password secret stored in Jenkins.

After all that, this worked and printed out the incoming password in plaintext, which is the flag we're after:

`ssh-mitm server --disable-publickey-auth --enable-keyboard-interactive-auth`
