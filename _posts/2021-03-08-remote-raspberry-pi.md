---
layout: default
title: Raspberry Pi - Remote Access
---
## Raspberry Pi: Remote Access

### Situation

So you have a Raspberry Pi or similar device on your home network and you want to connect to it via SSH from outside your network, through the internet?  There are several reasons you might want to do this, the main one is that you're running it as a server.  We will also discuss accessing an http server on the pi with similar methods, all for free.

The classic method involves some variation of the following:
1. Accessing your router to give the pi a static local IP address (which, it seems, doesn't really change much anyway, but could)
1. Create port forwarding rules so incoming connections over the internet to the router's IP address are forwarded directly to the pi
1. Using a service like [Duck DNS](https://www.duckdns.org/) to translate your possibly dynamic home network IP address to a static URL access point

The above method is fine except for two things which are really the same problem:
- You may not have access to your router, or permission, or may not want to raise eyebrows by messing with the router settings.
- Opening a port on your router like that presents security vulnerabilities.  In fact, the router your ISP provides may not even allow it.  Most of the time, requests on the internet originate from *inside* the network, but this time we're opening up the network to accept requests *outside* the network.  I think you can see how that's a problem.

Before anything, please visit [https://www.raspberrypi.org/documentation/configuration/security.md](https://www.raspberrypi.org/documentation/configuration/security.md) and follow the guidelines presented to secure your device.  I also created some software to help you with securing your pi: [https://github.com/rbrutherford3/initpi](https://github.com/rbrutherford3/initpi)

### More solutions

If you are using a VPN already, then your problem is effectively solved, because if both your pi and your SSH client are both operating on the same [virtual] network over the internet.  You may require additional configuration in order to connect, but I don't currently have a VPN, so I can't speak to this solution.  This solution has the benefit of an additional layer of security.

The second method is called a reverse ssh tunnel proxy and it works something like this:
1. Set up a server outside your network, maybe using a virtual private server (VPS) to accept SSH connections.
1. Set up your pi to connect as a client to this server, and establish a persistent connection.  Because it's initiating the connection instead of receiving it, your router doesn't mind.
1. Set up your server to accept TCP requests  and forward them to the pi through the reverse connection.
1. Set up your pi to forward the requests to local services.  So if you're connecting remotely to the pi via SSH, then the communication is actually SSH (client to server to pi) within SSH (pi to server to client).

So this is all well and good if you have an actual or virtual server at your disposal.  They do have services like AWS and GoogleCloud which are free, but they both require credit card information in case you exceed your limits, so I'm not interested.

### Third-party solutions:

This problem is apparently not uncommon, so much so that several services (some specific to Raspberry Pi) have evolved over the years.  I have divided them into two basic categories: encapsulated services and general services.  Encapsulated services connect, in their own way, to your pi and provide all sorts of tools like browser-based terminals and live vitals (CPU & RAM, temperature, etc).  They can also provide their own kind of tunneling, some more customizable over others.  Others allow no custom tunneling and instead require you to use their own tools.  General services are pretty basic, have no external software, and often require a little bit of messing with settings, etc to get things going the way you want.  The upshot is that they are more customizable.

#### [PiTunnel](https://pitunnel.com/) (encapsulated)

I'll start with PiTunnel because it's the only one made specifically for the pi and nothing else.  It has a browser-based terminal, custom tunnels for any TCP service, and live stats.  It's also a persistent connection and a dedicated HTTP address for web servers.  The drawbacks are: HTTP services are only a trial.  Also, the other custom connections are persistent, but the address changes every couple hours.  Finally, the browser terminal input lags noticeably.  This solution is ideal for beginners who also want SSH access.  If you don't need SSH access, move on to...

#### [Dataplicity](https://www.dataplicity.com/) (encapsulated)

Dataplicity is not just for the pi, but the pi is it's top consideration.  It has a browser based terminal, live stats, a dedicated HTTP tunnel they call a "wormhole," plus software and apps you can install to get the job done outside of the browser login.  The browser terminal is very responsive, unlike PiTunnel, but the drawback is the lack of customizable tunnels, so SSH is simply not possible through this service.  This solution is ideal for beginners also, who do not require any customization.

#### [remote.it](https://remote.it/)
Remote.it (formerly Waeaved) is a custom tunnel solution only.  What's unique about this service is that the tunnels are created through the website, not the pi, which is great if you can't access your pi, but bad for security, since a hacker could create a tunnel.  It has support for the Raspberry Pi, and you set up port forwarding rules through their website and they give you a temporary address.  The plus is the pi itself is always connected, so the temporary connections are always able to be created, but holy hell is it annoying how short they last, and using their site through a smartphone is also not fun.  This is an ideal solution if you only ever need a tunnel for a short time, but always need the option to be available.

#### [pagekite](https://pagekite.net/)

Pagekite is not designed for pis at all, but rather it's all custom tunnels, and a very good resource, too.  The pros are the simple URL you get (name.pagekite.me), custom subdomains, full customization, including the use of your own domain if you have one and the option to use your own SSL certificates.  The biggest con is that it's freemium, so if you are using it for light uses such as SSH and testing websites, you have to either pay $3/mo or fill out a form every month.  Also, this is more specific to me, but SSH connections use an HTTP proxy, and all the free SSH client apps on Android don't have that feature.

#### [Openport](https://openport.io/)

Openport is a service that creates a port on their domain and forwards that port to a specific port on your pi.  Accessing their port requires IP validation by the user via a generated temporary URL, which then sends them to a page with the port link, which you can then use to connect via any TCP protocol.  Free account limitations include 1 machine, 1 connection, 1 day at a time (24 hour limits).  The connection lags, and the setup process is kind of absurd just in sheer number of steps, just to get a temporary tunnel.  I do not recommend this option to anyone, even if you're just using it to temporarily share access with someone you trust, which is all it's really good for.  Plus, how are you going to re-set up the connection if you don't have access to the pi in the first place?

#### [ngrok](https://ngrok.com/)

This service is not just for raspberry pis, but it’s ideal for them.  ngrok is free but has paid services for more connections.  It has a very simple installation: just download their file, create a configuration file, and start it up.  Unlike other services where the free option kicks you off and changes names often, ngrok URLs last for weeks, so you’re not constantly creating new tunnels.  It allows one TCP tunnel and one HTTP tunnel.  You can also create your own service for ngrok that will start from boot and restart if forced to stop, automatically.  It is also a very reliable connection while you’re on it.  Other than the lack of custom domains, the only downside of ngrok is that it disconnects fairly quickly due to inactivity while on an SSH session.

#### Others
Here is a lsit of other services that I did not rate for various reasons, but they mostly follow ngrok’s model:

- [SSHReach.me](https://sshreach.me/): not a free service, free trial only
- [serveo](http://serveo.net/): popular, was temporarily disabled, may be insecure
- holepunch.io: wasn't able to access website
- [localhost.run](https://localhost.run/): looks promising
- [localtunnel.me](https://theboroer.github.io/localtunnel-www/): didn't try
- [inlets](https://inlets.dev/): didn't try

