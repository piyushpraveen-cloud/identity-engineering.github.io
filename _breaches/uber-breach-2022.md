---
title: "The Uber Breach of 2022: One Tired Employee and One Careless Script"
date: 2026-10-12
author: Piyush Praveen
tags: [breach, privileged-access, mfa, pam, uber]
summary: How an attacker used a stolen password and a script on a file share to reach the heart of Uber's systems.
---

## Overview

In September 2022 Uber confirmed that an attacker had gained deep access to its internal systems. The attacker even posted a message in the company Slack. Staff saw it and the incident became public very quickly.

What makes this breach worth studying is how simple it was. There was no clever exploit. The attacker used a stolen password and a tired employee. Then they found a script that held the keys to everything. I work in privileged access every day and I see this as a textbook case of why identity is now the real perimeter.

## What happened

**1. A contractor's login was stolen.** Uber said a contractor's personal device was infected with malware. The contractor's login details were then sold on the dark web.

**2. The attacker wore down the second factor.** The attacker kept trying to log in with the stolen password. Each attempt sent an approval request to the contractor's phone. This is often called MFA fatigue or push bombing. The attacker then contacted the contractor on WhatsApp and pretended to be Uber IT. They said the requests would stop if the contractor approved one. The contractor approved it.

**3. The attacker got onto the network.** With a valid login and an approved prompt the attacker connected through the VPN as a normal user.

**4. A script gave away the keys.** On the internal network the attacker found a share holding PowerShell scripts. One script contained the admin credentials for Uber's privileged access management tool. This is the most painful part of the story. The tool exists to protect admin passwords and one of them was sitting in plain text.

**5. Access spread fast.** From the PAM tool the attacker reached many other systems. Public reports named cloud platforms and identity and security tools. They also reached Slack and the internal bug reports that researchers had submitted.

**6. The attacker announced themselves.** The message in Slack ended any hope of a quiet response. Uber took systems offline and locked down access while it investigated.

Uber linked the attacker to the Lapsus$ group. The attacker was reportedly a teenager. Uber said it saw no sign that the attacker reached the production systems that run its apps or changed its code.

## Why it worked

Three weaknesses lined up.

**Weak MFA.** A simple approve button can be abused by anyone who has the password. People get tired and people want the noise to stop.

**Hardcoded privileged credentials.** A secret inside a script is a secret anyone with read access can take. It also never gets rotated in practice because nobody remembers it is there.

**Too much reach.** A contractor login should not lead to a share full of scripts. It certainly should not lead to the admin account of the PAM platform. Once that account was taken every vault protection became meaningless.

## My view as a PAM practitioner

A vault is only as strong as the weakest place its password is stored. We spend months onboarding accounts and setting rotation and session recording. All of that falls apart if the credential that controls the vault sits in a script.

This is also a reminder that PAM is not only a product. It is a habit. Every script and pipeline and scheduled task needs a safe way to get a secret at the moment it needs it. That means an API call to the vault and not a password pasted into the code.

## What I would do differently

- **Use phishing resistant MFA.** Number matching is a minimum. FIDO2 keys or passkeys are better.
- **Limit and alert on repeated prompts.** Ten requests in a few minutes should lock the account and raise an alert.
- **Scan for secrets.** Search shares and repositories and scripts for passwords and keys. Remove what you find and rotate it.
- **Protect the PAM admin accounts most of all.** Use separate admin identities and restrict where they can log in. Alert on every use.
- **Apply least privilege to contractors.** Give them access to what they need and nothing else. Review it often.
- **Segment the network.** A VPN login should not give a view of the whole estate.
- **Practise your response.** Know how to cut access and rotate secrets at speed.

## Key takeaways

1. MFA that relies on a single tap can be beaten by persistence.
2. Never store a privileged credential in a script or on a share.
3. The accounts that guard your vault need the strongest protection you have.
4. Contractors and third parties are part of your attack surface.
5. Good security is mostly about the basics done every time.

Most of the lessons here are not new. That is the point. Breaches like this one remind us to check the basics again.

*This article is based on Uber's public security updates and press coverage from September 2022. The views are my own.*
