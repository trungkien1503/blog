---
layout: post
title: Getting started with Rails on Docker
---
Today, it's the first time I buy a domain for my website. After searching, I choose GoDaddy.

If you do not know that how to buy a domain,

Please check this link: [how to buy a domain][1]

I plan to connect my domain from Godaddy and my Heroku app.

Alright! It works! Here is how I did it from scratch, so others with the same problem can fix it too.

First, I will explain how to setup Heroku and GoDaddy, then I will explain how to create a naked domain (www.example.com -> example.com).

# Setup Heroku and GoDaddy:

*   In your project folder in terminal (on your computer) write `heroku domains:add www.example.com` (where [www.example.com][2] is the domain you have bought at GoDaddy)

*   Sign in to GoDaddy -> DOMAINS -> choose your domain -> Launch (this will take you to the Domain Details)

*   Click 'DNS Zone File' tab

*   Remove the CNAME record named 'www' (which points to @)

*   Click 'Add record' -> CNAME(Alias) -> 'Host' should be www and 'Points to' should be your Heroku address (example supermoo-bil-3411.herokuapp.com). TTL can be 1 hour.

*   It can take some time for the DNS to propagate. It took me about 10 minutes.

*   That's it! [supermoo-bil-3411.herokuapp.com][3] will now be under [www.example.com][3] :)

# Create a naked domain:

A naked domain removes the need to write www in front of your domain name. This can be done by forwarding example.com to www.example.com. This is super easy on GoDaddy:

*   In the same window as above, click on the 'Settings' tab

*   Under Forwarding -> Domain -> Click 'Manage' -> then click 'Add Forwarding'

*   'Forward to' should be www.example.com (your domain), 'Redirect type' should be '301 (Permanent)', 'Forward settings' should be 'Forward only'

*   Make sure "Update my nameservers and DNS settings to support this change. (Recommended)" is checked

That's it! You are done :)

## Useful links:

<https://devcenter.heroku.com/articles/custom-domains>

Thanks to Ryan Kazinec and Allegutta for help :)

 [1]: http://thachpham.com/hosting-domain/mua-ten-mien-godaddy.html
 [2]: http://example.com
 [3]: http://