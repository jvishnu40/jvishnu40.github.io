---
title:      SQL for Dummies
layout:     post
category:   Notes
tags: 	    [beginner, TLS, SSL]
---

Its 2025, and we share everything from political memes to bank detials online. SSL (Secure Sockets Layer) is like the invisible hero that is keeping your date safe. It’s that little padlock in your browser that quietly works behind the scenes to protect your info from digital snoops. That is the internetes way of saying, Ddont worry, you may be stupid but I am not."

<!--more-->

## What is SSL
 SSl is Secure Sockets Layer. It is an encryption based internet security based protocol. A website that has SSl/TLS implemeneted has hhtps instead of http in the browser URL. 


## Why do we need SSL/TLS
We need SSL/TLS because when data is transmitted accross the internet it could be intercepted by a bad person. Lets say if in any case, you say you will need send PII on the internet but i dont see how that is possible but lets just assume that is possible. Since you are not sending PII, you may this that not using SSL doesnt pose any risk for you. This is where you are absolutely wrong. The content itself might not be sensitive but risk is sstill high. this is because one may intercept and even alter the contect you include a payload that maybe a malware that th receiving end might receiving. They may even trakc you online activity or even steal cookies to hijack your accounts.

## What are SSL certificates?
SSL can be implemented if a website has a SSL/TLS certificate. A ceritificate primarily contains a public key & a private key. From the user end (user's browser) the data is encrpypted using the public key and it is decrypted on the server end using the private key. Which server, you might ask? It is the server that is hosting your website that you're accessing. 

## What are the types of certificates? 

| Certificate      | What is it? | 
| :---:       |    :----:   | 
| Single-domain      | a cert that applies to only one domain like almanack.vishnu.lol       |
| Wildcard domain    | Like a single domain but this guy is applied to the single domain and its subdomain. The name translates to something like *.vishnu.lol. These domain are inlcluded vishnu.lol, hello.vishnu.lol, yolo.vishnu.lol       |
|multi domain| Used for multi domain but it is set to those domain only. Think bundle cert. |

## What are certficate validations?

When we buy a certificate we will need to validate that the certificate. Why this is needed is because it is to confirm that we own the domain and the cert is valid.

|Validation methd|Explaination |
|:---:|:---:|
|Domain Validation|lThe least stringent and cheapest option. The certificate authority (CA) only verifies domain ownership, typically through a DNS CNAME record, email verification, or an HTTP challenge. No business identity verification is done.|
|Organization Validation |The CA verifies both domain ownership and the organization requesting the certificate. They contact the organization via email (e.g., admin@vishnu.lol) and may require additional documentation to confirm legitimacy. This provides more trust than DV.|
|Extended Validation | The highest level of verification. The CA performs a full background check on the business, including legal, operational, and physical existence. EV certificates are often used by financial institutions and high-trust websites. They used to display a green address bar in browsers but now mainly show the organization’s name in the certificate details.|
