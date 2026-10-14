---
layout: post
title: "Independent Breakglass for Critical Access"
date: 2026-10-14
tags: [breakglass, privileged access, PAM, CyberArk, M-of-N, whitepaper]
summary: "A simple guide to building an emergency access process that does not depend on the network or on CyberArk or on any one person."
---

*A whitepaper for security leaders and engineers*

## 1. Executive Summary

Every company needs a last resort way into its most critical systems. This is called breakglass. It is used when normal access has failed. This can happen during an outage or a cyber attack.

Many companies keep their breakglass passwords inside their privileged access tool. This is a risk. If the tool fails then the passwords fail with it.

This paper explains how to build a breakglass process that does not depend on the network. It does not depend on CyberArk either. It also does not depend on any one person. The design uses offline storage and shared control. It uses a method called M-of-N. It adds strong monitoring and a clear process. It also needs regular testing.

## 2. The Problem

Normal admin access relies on many things. These include the network and DNS and the directory and the identity provider and multi-factor authentication and the privileged access vault. If one of them fails then access can fail. If several fail together then you can be locked out of your own company.

Attackers know this. A ransomware attack often hits the directory and the backups at the same time. At that moment the team needs access that does not rely on those systems.

A weak breakglass process often has these faults:

- The passwords are kept in the same vault that they protect.
- One person holds the only copy.
- Nobody is alerted when the account is used.
- The process is never tested.
- The password is not changed after use.

## 3. Design Principles

1. **Independent.** Breakglass must not rely on any system that it helps to recover.
2. **No single point of failure.** No one person or place or device can block access or grant it alone.
3. **Shared control.** Access needs a group of people. This uses M-of-N.
4. **Alert on every use.** Alerts must travel by paths that do not share a failure point with the target system.
5. **Use means change.** Every password is changed after use.
6. **Test often.** A plan that is never tested is not a plan.

## 4. What to Protect

Breakglass is more than one admin password. List every critical asset that you need to recover your company.

| Area | What to include |
|---|---|
| CyberArk | Vault Master CD with Master and Recovery keys and the Operator CD and the Server key and the Vault admin passwords and the DR Vault credentials |
| Active Directory | Two emergency admin accounts and the DSRM password for each domain controller and the KRBTGT recovery plan |
| Cloud identity | Two cloud-only Global Admin accounts and their FIDO2 keys |
| Cloud root | AWS root or Azure owner or GCP org admin with hardware MFA |
| Network and security | Firewall and switch and VPN and EDR console admin accounts |
| Infrastructure | Hypervisor root and storage and backup admin and HSM cards and the PKI root |
| Outside suppliers | Domain registrar and DNS provider and carrier portals and certificate authority |
| Out-of-band access | Console servers and iDRAC and iLO credentials |

If the Vault is lost and nobody holds its recovery keys then everything inside it is lost. This is why the Vault recovery material is in scope.

## 5. How the Solution Works

### 5.1 Creating the Credentials

All credentials are made in a planned ceremony. It follows these rules:

- Use a clean laptop that is not on any network. Build it fresh from a trusted source.
- Have at least three people present. These are a security engineer and an infrastructure owner and an independent witness from Audit or Risk.
- Record the event on video. Add a scribe to write a log.
- Use a secure random generator. Make admin passwords 24 to 32 characters long. For passwords that people may need to type under stress use six or more random words.
- Make every credential unique.
- The ceremony log records the credential name and the date and the seal number. It never records the secret.
- Wipe the laptop when the ceremony ends.

### 5.2 Understanding M-of-N

M-of-N is a rule for shared control. A secret is split into N parts called shares. Any M of those shares can rebuild the secret. Fewer than M shares reveal nothing at all.

**Example.** In a 3-of-5 set up the key is split into 5 shares. Five custodians each hold one share. Any 3 of them can unlock the vault. One or two shares are useless on their own.

| Risk | How M-of-N helps |
|---|---|
| Insider misuse | No one person can reach the secret alone |
| Absent custodian | With 5 holders and a threshold of 3 up to 2 people can be away and access still works |
| Lost or stolen share | One share reveals nothing and new shares can be issued |
| Social engineering | An attacker must fool several people in different places |

**Choosing M and N.** If M is too low then a small group can work together to break in. If M is too high then one absence can block emergency access. Most companies choose 3-of-5. Larger companies may choose 4-of-7. Keep M just above half of N. Keep the gap between N and M at 2 or more.

**How it is done.** The standard method is Shamir's Secret Sharing. Open source tools such as ssss can do it. HashiCorp Vault uses it for its unseal keys. Many hardware security modules offer similar quorum controls with smartcards. Dual control such as two people for one safe is a simple form of the same idea.

### 5.3 Storing the Credentials

Use two tiers together.

**Tier A. Offline encrypted vault.**

- Put all breakglass credentials into one encrypted file. A KeePassXC database or an age or GPG file works well.
- Keep the file on hardware encrypted USB drives.
- Split the master key into 3-of-5 shares using Shamir.
- Put each share on its own medium such as a metal card or an encrypted USB.
- Seal each share in a numbered tamper-evident bag.
- Keep copies of the encrypted vault in at least three places.

**Tier B. Sealed paper credentials.**

- Print each critical password on a printer that is not on the network.
- Seal each one in a numbered tamper-evident envelope.
- Keep the envelopes in rated safes with two locks that need two people.
- Use three locations. These are the head office secure room and a disaster recovery site in another region and a bank vault or third party escrow that only a named group can open.

**Never** store breakglass credentials in CyberArk or a password manager or a ticket or an email or a cloud drive.

### 5.4 Choosing Custodians

- Choose five to seven custodians from different teams. Include Security and Infrastructure and IT leadership and Risk or Audit and Legal or an executive.
- Name a deputy for each custodian.
- Place custodians in at least two regions.
- If a custodian leaves or changes role then make new shares. Do not just hand over the old one.

### 5.5 Breakglass Laptops and Out-of-Band Access

- Keep two prepared laptops at two different sites.
- Do not join them to the domain. Encrypt the full disk. Keep the software list short.
- Give each laptop a mobile data connection so it works when the company network is down.
- Keep the laptops switched off in the safes. Update and rebuild them every quarter.
- Set up a small out-of-band management network on a different internet line or mobile link. It reaches the console servers and firewalls and hypervisors without using the main network.

## 6. Account Hardening

**Active Directory**

- Create two dedicated accounts in a locked OU.
- Never use them for daily work or for services. Give them no mailbox.
- Allow logon only on domain controllers and the breakglass laptop.
- Use passwords only. Do not rely on smartcards or PKI or MFA that may be down.
- Make sure password expiry cannot disable them without notice.
- Give every domain controller its own DSRM password.

**Entra ID or Okta**

- Create two cloud-only accounts on the default cloud domain. Do not federate or sync them.
- Use a different strong sign-in method for each account. For example one can use FIDO2 keys and the other a certificate. A failure in one method then cannot lock you out.
- Keep the FIDO2 keys apart from the passwords.
- Microsoft advises that emergency accounts are left out of Conditional Access rules that could lock the tenant. Make up for this with strong sign-in and close monitoring.

**Cloud root accounts**

- Use a long unique password and a hardware MFA key. Seal both in at least two places.
- Raise an alert on any root sign-in.

**CyberArk**

- Store the Master CD and Recovery key material in the same offline system.
- Practise a full Vault recovery at least once a year.

## 7. Monitoring

Breakglass use should be rare and loud. Any sign-in is a top priority alert until it is proven to be valid.

### 7.1 Digital Monitoring

- **Alert rules.** Alert on any sign-in by a breakglass account. Use Windows events 4624 and 4625 and 4768 and 4769 and 4776. Use Entra sign-in logs. Use AWS root events.
- **Change alerts.** Alert on any password reset or group change or disable or delete on these accounts. Alert on changes to Conditional Access rules that affect them.
- **Separate alert paths.** Send alerts by SMS and voice call and a pager tool and a cloud-native alert and the SIEM. Do not rely on company email alone.
- **Wide audience.** Send alerts to at least five people including the CISO and Internal Audit. Nobody can then hide an alert alone.
- **Safe log copy.** Send logs to write-once storage in a separate account or tenant.
- **Heartbeat check.** Run a scheduled test that confirms each account still exists and is enabled and has the right password state. Alert on any change.

### 7.2 Physical Monitoring

- Fit door sensors and keep access logs and cameras at every safe location.
- Check the tamper-evident seals every month or every quarter. Match each seal number to the register. Two people sign the check.

### 7.3 Matching Use to Records

Every sign-in must match a written breakglass record. A sign-in with no record is a security incident. A broken seal with no sign-in is also a security incident.

## 8. The Breakglass Process

**Step 1. Declare.** Use breakglass only for clear reasons. These are a Tier-0 outage or CyberArk and identity provider failure or a major incident such as ransomware or a total loss of normal admin access. Any trained Incident Commander can declare. Keep at least three trained people in this role.

**Step 2. Approve.** Two approvers from a set list give approval by phone or by the printed contact sheet. This must not rely on company email or chat or single sign-on.

**Step 3. Retrieve.** At least two custodians and one witness go to the nearest safe and follow the checklist. For the Tier A vault three custodians join their shares on the breakglass laptop. Write down the seal numbers and the time and the names and the reason.

**Step 4. Use.** Work from the breakglass laptop. Keep two people present. One works and one watches and records. Record the screen or keep a full command log. Set a time limit such as four hours. Extend it only with new approval. Use the lowest level of account that can do the job.

**Step 5. Notify.** The automatic alert fires on sign-in. The Incident Commander also tells the CISO and Audit by hand as a back up.

**Step 6. Close.** Sign out. Rebuild the laptop. Return or destroy used papers. Go back to normal access as soon as the main systems return.

**Step 7. Change and reseal.** Change every credential that was taken out within 24 hours. Do this even if you think nobody typed it. Hold a short new ceremony. Make new sealed envelopes and update the vault copies in every location. If shares were used then consider making new shares.

**Step 8. Review.** Hold a review within 72 hours. Ask what was done and whether it was needed. Ask if the alerts and the retrieval and the password change all worked. Send findings to Risk and Audit.

## 9. What Happens When Things Fail

| Event | How the design copes |
|---|---|
| CyberArk Vault fails | The credentials are not inside it. The recovery keys are held offline. |
| Network is down | The laptop uses mobile data and the out-of-band network. Paper credentials need no network. |
| Directory or identity provider is down | Cloud-only accounts and local accounts still work. Alerts use separate paths. |
| One custodian is away or leaves | The M-of-N rule and the deputies keep access open. New shares are made later. |
| Head office is lost | Copies are kept at the recovery site and the escrow. |
| Insider tries to misuse access | Shared control and wide alerts and seals and cameras stop or expose it. |
| Custodian is tricked by a caller | Approval needs two people and a witness by a second route. |
| SIEM is attacked | Cloud-native and SMS alerts and safe log copies still exist. |
| All custodians are unreachable | The escrow vault opens with an executive group as a last resort. |
| Account used in secret | Alerts on any use and the heartbeat check expose it. |

## 10. Governance and Testing

- **Policy.** Write a policy that the CISO and CIO and Risk approve. It names owners and custodians and rules for use and review duties.
- **People check.** Review custodians and deputies every quarter and after any staff change.
- **Monthly.** Check seals and run the heartbeat test.
- **Quarterly.** Run a talk-through exercise and test every alert path with a drill account.
- **Yearly.** Run a full live test with the real credentials. Change them afterwards. Practise Vault recovery in the same year.
- **Planned change.** Change the credentials once a year even if unused. Change them after any custodian change.
- **Measures.** Track the time from declaring to access. Aim for under 60 minutes. Track alert delivery time and the share of seals checked and the test findings closed.
- **Audit.** Ask for an independent review each year and report issues to the risk committee.

## 11. Common Mistakes

1. Keeping the breakglass password in the same vault that it backs up.
2. Needing MFA from the system that is down.
3. Sending alerts only to email on the failed tenant.
4. One person holding the only copy.
5. Expired passwords or disabled accounts that nobody noticed.
6. Not changing the password after use.
7. Never testing so that the first real use is the first practice.
8. Using breakglass accounts for easy daily admin work. This destroys the clear alert signal.

## 12. Roadmap

| Phase | Time | Result |
|---|---|---|
| Foundations | Weeks 1 to 2 | Asset list and policy and custodians and locations |
| Build | Weeks 3 to 6 | Hardened accounts and ceremony script and Shamir set up and safes and laptops and out-of-band path |
| Detect | Weeks 4 to 7 | Alert rules and separate alert paths and safe log copy |
| Ceremony | Week 8 | Credentials made and sealed and shared and placed in escrow |
| Validate | Weeks 9 to 10 | Talk-through and live test and fixes and sign-off |
| Operate | Ongoing | Checks and reviews and changes and yearly audit |

## 13. Glossary

| Term | Meaning |
|---|---|
| Breakglass | An emergency way to get admin access when normal access fails |
| Custodian | A trusted person who holds one share or key |
| Share | One part of a secret that has been split |
| Quorum | The smallest group needed to act |
| Tamper-evident | Made so that any opening leaves a clear sign |
| Out-of-band | Using a path that is separate from the main network |

## 14. Conclusion

A good breakglass process is simple to explain and hard to break. Keep the credentials offline and split them with M-of-N. Spread them across places and people. Watch every use with alerts that do not share a failure point. Change everything after use and test the whole process often. With these steps your company can regain control in a crisis without trusting any one person or any one system.
