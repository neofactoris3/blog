---
layout: post
title: SSH from the internet without port forwarding
permalink: ssh-from-the-internet
---

If you do not want to open ports on your network but want to ssh from the internet you could use Cloudflare Access to achieve this.

![Cloudflare Access](/assets/images/cloudflare0.png "Zero Trust")

## create App on Cloudflare Access

Head to the Zero Trust dashboard to create a new application. Select the Applications page from the sidebar. Click **Add application**.

Select *Self Hosted*

![Cloudflare Access](/assets/images/cloudflare1.png "Add an application")

Enter the *Application name* and add the subdomain:

![Cloudflare Access](/assets/images/cloudflare2.png "Application name")

Edit the App Policy to allow only the email address that you would like to authenticate:

![Cloudflare Access](/assets/images/cloudflare3.png "Restrict to specific emails")

***That’s it!***

## install cloudflared on the origin server
Cloudflare Tunnel creates a secure, outbound-only, connection between the origin server and Cloudflare’s network. With this, you can lock down any externally exposed points of ingress which means no open ports. So, now we want to configure the tunnel on the origin server.

For that, grab the package from <a href="https://github.com/cloudflare/cloudflared/releases" target="_blank">Github</a>

Install the downloaded deb (in case if it is a Debian derivative):

`sudo dpkg -i ./cloudflared.deb`

Run the following command on the server to authenticate cloudflared into your Cloudflare account.

`cloudflared tunnel login`

*cloudflared* on the headless server will let you copy the URL from the command-line output so that you could visit the URL in a browser on any machine and authenticate.

## creating the tunnel

`cloudflared tunnel create <tunnel name>`

This will create a tunnel for you and output the relevant details to proceed to the next step:

Tunnel credentials written to */home/user/.cloudflared/ce347-ewr-rbweb-b1b.json*. *cloudflared* chose this file based on where your origin certificate was found. Keep this file secret. To revoke these credentials, delete the tunnel.

Created tunnel `<tunnel name> with id <unique id>`

Create a YAML file that cloudflared can reach. By default, cloudflared will look for the file in the same folder where cloudflared has been installed.

`nano ~/.cloudflared/config.yml`

Next, configure the Tunnel, replacing the example ID below with the unique ID of the Tunnel created above. Additionally, replace the hostname in this example with the hostname of the application configured with Cloudflare Access.

*tunnel*: `6ff42ae2-id-demo-unique-31cid1551ef`  
*credentials-file*: `/home/user/.cloudflared/6ff42ae2-765d-4adf-8112-31c55c1551ef.json`

ingress:  
- *hostname*: `ssh.yourdomain.com`  
- service: `ssh://localhost:<ssh port>` 
- service: `http_status:404`

You can now create a DNS record that will route traffic to this Tunnel. Multiple DNS records can point to a single Tunnel and will send traffic to the service configured as long as the hostname is defined with an ingress rule.

Navigate to <https://dash.cloudflare.com> and choose the hostname where you want to create a Tunnel. This should match the hostname of the Access policy. Click **+ Add record**.

Select **CNAME** as the record type. For the target, input the ID of your Tunnel followed by cfargotunnel.com. In this example, the target would be:

**\<unique id\>***.cfargotunnel.com*

## run cloudflared as a service

I use linux so, in this article I will explain how to do it via systemctl.

`sudo mv /home/user/.cloudflared/config.yml /etc/cloudflared/`  
`cloudflared service install`  
`sudo systemctl enable cloudflared`

finally…

`sudo systemctl start cloudflared`

## SSH-ing from a browser on the client

Cloudflare can render an SSH client in your browser.

To enable, navigate to the application page of the Access section in the Zero Trust dashboard. Click **Edit** and select the Settings tab. In the ***cloudflared settings card***, select ***SSH*** from the ***Browser Rendering*** drop-down menu.

Once enabled, when users authenticate and visit the URL of the application, Cloudflare will render a terminal in their browser.

![Cloudflare Access](/assets/images/cloudflare4.png "SSH from browser terminal")

### ***Thank you!***