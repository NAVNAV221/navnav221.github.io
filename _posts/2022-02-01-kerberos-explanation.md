---
layout: post
title: About Kerberos
subtitle: Explanation of Kerberos protocol and the related attacks
categories: Windows Protocols
cover-img: /assets/img/kerberos/cerberus_dogs_cover_img.png
thumbnail-img: /assets/img/kerberos/cerberus_dogs_wallpaper.png
tags: [Kerberos, ASREProast, Pass The Ticket, Golden Ticket, Silver Ticket, Kerberoasting, Impacket]
comments: true
---
First of all, have you ever wondered why the name "kerberos" chosen? well, meet the guards of the underworld:![image-20220713225240113](/assets/img/kerberos/cerberus_wiki_explanation.png)

Yes, the Greek mythology states that ‘Cerberus’ is a three-headed dog that guards the gates of the “underworld” ...

**What is the kerberos protocol?**

Authentication protocol in domain environment intended to authenticate users against domain's services.

Kerberos provide a third pary entity called KDC which improve the authentication process and prevents MITM attacks and imperonating attacks.

The KDC consists from the following compnents:

- AS (Authentication Server)
- TGS (TIcket Granting Server)

This is the entity that responsible for the authentication process and issues the tickets.

The structue looks like the picture below:

![image-20220713225737635](/assets/img/kerberos/kerberos_dogs_illustration.png)

(ALL THE IMAGE DETAILS WILL EXPLAIN LATER)

<u>Client</u> - First Dog, <u>KDC</u> - Second Dog, <u>Services</u> - Third Dog

# Authentication Vs. Authorization

Imagine you visit Facebook, in the lobby you represent your ID card to get a magnet card that will serve you along the tour: food area, game room, etc… You enjoy the tech view of the company and how cool the place is, and boom! you see on your left the server room of facebook - cool! You are so curious and you want to get in, you try to use your magnet card to unlock the door and the door isn't open → why? because it's allowed for Facebook staff only.

**<u>Authentication Process:</u>**
In the lobby you need to represent an ID card so the staff could determine whether you are who you claim you are and then they’ll activate a magnate card for your tour.
<u>**Authorization Process:**</u>
Your magnate card didn't work at the server room because you didn’t have access to the server room (you weren’t one of the Facebook staff).

**In simple terms, authentication is the process of verifying who a user is, while authorization is the process of verifying what they have access to.**

# The process :footprints:

## KRB_AS_REQ

Alice send a KRB_AS_REQ to the AS, the direct translate is: "Hey AS, Im Alice and I want to get a TGT from you". Ticket Granting Ticket (TGT) is the ticket that Alice can present to the TGS and request ticket for the wanted service. **Part of the request encrypted with the user secret key (C) that consist Alice NTLM hash password**. The encrpytion intended for the AS to verify that it is indeed Alice.

(Im deliberately dosen't include here the packet's content because it's distracting from the essence of the protocol).

![image-20220723100654193](/assets/img/kerberos/KRB_AS_REQ.png)

## KRB_AS_REP

Now the AS gets the request and verify that he talk with Alice. So he retrive Alice NTLM hash password (Kc) and decrypt the encrypted part. If he success he'll send to Alice the following things inside the packet:

- TGT encrypted with the **TGS secret key (Ktgs)** (Yes, only the KDC can decrypt it).
- **Session key** (Kc-tgs) that Alice would use to talk with the KDC next time. This key encrypted with the **user secret key** so only Alice and the KDC can read it.

![image-20220723103243443](/assets/img/kerberos/KRB_AS_REP.png)

## TGS_REQ

After that Alice gets the TGT, she'll send it to the TGS to request the service ticket to the target service (AP - Application Server). The request include the following:

- TGT.
- Authenticator that verify Alice (authenticator content explained later).
- Target service info.

![image-20220723101401588](/assets/img/kerberos/TGS_REQ.png)

## TGS_REP

The TGS open the TGT with the **TGS secret key** to verify he valid. Also he verify the **authenticator** with the **user secret key** (that he retrive from the DC) to ensure he belongs to Alice.

Now the TGS prepare the "entrance ticket" (service ticket) to the target server (AP). The **service ticket** include:

- Service name.
- Alice details.

This ticket encrypted by the **Service secret key (** - The key that only the KDC and the target server (AP) knows. The encryption include the AP NTLM Hash password.

Also, the response packet include **a new session key!** thah include the **user secret key** (recall that the TGS and the AS is two separate entities).

![image-20220723102252972](/assets/img/kerberos/TGS_REP.png)

## AP_REQ

Now that Alice has the **Service ticket** to the AP, the AP_REQ Alice will send include the **service ticket encrypted with the service secret key** and the **authenticator that encrypted with the user session key** (The one from the TGS_REP).

![image-20220723103728676](/assets/img/kerberos/AP_REQ.png)

## AP_REP

The AP now sends a timestamp encrypted with the server session key so Alice verify that it is indeed the target service she request.

![image-20220723104017106](/assets/img/kerberos/AP_REP.png)

## All in one shot

![image-20220723104025480](/assets/img/kerberos/ALL_PROCESS_IMAGE.png)

# Packet's content in brief :envelope:

#### Reptitve fields

- Padata (Pre-Authentication Data): This field encrypted with Alice's password and include a timestamp. This field intended to protect from replay attacks so attacker couldn't replay a request in the future after the user authenticated.
- Cname - Client username.
- Realm - Domain name.
- Sname - The requested service name.
- Tiil - Wanted expiration time of the requested ticket.
- Etype - Supported encryption types.

## KRB_AS_REQ Packet

1. padata.
2. Cname - Alice username.
3. Realm - Domain name.
4. Sname - KDC server name (SPN).
5. Till - Wanted experation date of the TGT.
6. Etype - Alice supported encryption types.

## KRB_AS_REP

1. Crealm - Domain name.
2. Cname - Alice.
3. TGT Encrypted with the TGS secret key.
4. Session key encrypted with the **user secret key**.

## TGS_REQ

1. Padata.
2. Authenticator - Include timestamp, client identity, request properties (encryption type), request info (encryption details). All encrypted with the **user secret key**.
3. TGT.
4. Req-body - Include domain name (realm), target service name (Sname), supported encryption type (etype).

## TGS_REP

1. Padata.
2. Service ticket - encrypted with the **Service secret key**.
3. Session key - A new session key encrypted with the **user secret key**.

## AP_REQ

1. TGS encrypted with the **Service secret key**.
2. Authenticator.

## AP_REP

1. Timestamp encrypted with the **service secret key**. Intended for Alice to verify the AP is correct.

If you want the specific packet's fields so go and open Wireshark. Don't be lazy!

# Kerberos attacks :man_technologist:

## Overpass the Hash/Pass The Key (PTK)

[Required: User Hash]
If the attacker sniffs the hash of a user he can impersonate him against the KDC and gain access to several services.
<u>Where are those hashes located?</u> SAM files, NTDS.DIT file on DCs, lsass process via Mimikatz (Read the article about LSASS in the Infrastructure section) and it is also possible to find cleartext passwords.

## Pass The Ticket (PTT)

[Required: Ticket (TGT & TGS) & Session Key.]
Because of Kerberos TCP & UDP communication, attackers can easily read the data passed to each side by MITM attacks such as Arp Spoofing.
Well, session keys are an integral part of PTT attack and MITM attacks are not the way to get them, but an attacker can obtain them via the memory with Mimikatz.
Which ticket do you prefer to sniff, TGT or TGS? Think about it, how many service access do you want? access to only one service(TGS)? or to many (TGT)?

## Golden Ticket & Silver Ticket

The attacker builds a TGT with the NTLM hash of the ‘krbtgt’ account via Mimikatz.
Note: The Golden Ticket - TGT you create can be invalid if its expired or the ‘krbtgt’ account changes its password.
Silver Ticket is the same but you build a TGS instead. Here the service key is required, which is derived from the service owner account.
[NOTE: If the server verifies the PAC, you have to include the krbtgt key otherwise the attack won’t succeed].

## Kerberoasting

As we mentioned above, the TGS service ticket comes encrypted with a service key, which is derived from the service/server owner account’s NTLM hash. So, once we get it we can crack its password if he had an easy one.
As we know, Kerberos is an authentication protocol not an authorization. So we can take advantage of that and even with normal domain users we can get any TGS service key we want.
https://gist.github.com/TarlogicSecurity/2f221924fef8c14a1d8e29f3cb5c5c4a

## ASREPRoast

Like Kerberoasting, ASRERoast crack password (now the user one). BUT is take advantage of DONT_REQ_PREAUTH flag (Removes the encrypted timestamp from the user requests) when is set and then the attacker can built a KRB_AS_REQ message without specifying its password → The KDC with the KRB_AS_REP response message will contain encrypted information with the user key → The password can be cracked.
References
https://www.tarlogic.com/blog/how-kerberos-works/
https://www.sciencedirect.com/topics/computer-science/ticket-granting-service

# Kerberos & HTB

Impacket:

1.	Get server’s SPN - A service principal name (SPN) is a unique identifier of a service instance.
2.	Get user’s TGT that use DONT_REQ_PREAUTH flag - GetNPUsers.py
3.	Cheatsheet- https://gist.github.com/TarlogicSecurity/2f221924fef8c14a1d8e29f3cb5c5c4a.
4.	https://www.hackingarticles.in/abusing-kerberos-using-impacket/
https://cobalt.io/blog/kerberoast-attack-techniques

# Terminology

## Kerberos Realm

Is the group of systems which kerberos has the authority to authenticate a user to a service - file service or application. And in simple words, is where kerberos take action.

## Agents (The dogs)

These are the entities who work together to define the authentication process in Kerberos protocol.

### Client - Dog 1

The client/user who wants to access the service.

### Key Distribution Center (KDC) - Dog 2

The **main service** that is responsible for:

1. Issuing the tickets (explained in next section).
2. Generate temporary **session keys** which allow a user to securely authenticate to a service.
3. Store all the secret symmetric keys for users & services.

This service installed on the DC (Domain Controller)
It is supported by the **AS (Authentication Service)** → The service that issues the TGTs (explained in the next chapter).

### Application Server (AP) - Dog 3

Offers the required service by the user.

## AS (Authentication Server)

The AS receives a request containing the **username of the client** requesting authentication and **returns an encrypted <u>Ticket Granting Ticket</u> (TGT) for that user**. Then the client can use this TGS ticket to further steps and perform requests to other services.
●	**Granting the initial ticket** to the Ticket Granting Server.
●	Kerberos cached those tickets for **8-10 hours**.

## TGT (Ticket Granting Ticket)

Is the ticket presented to the KDC to request for service tickets. It’s encrypted with the **TGS key**.
**<u>KDC or krbtgt key**</u> - which is derivative from krbtgt account NTLM hash.

## TGS (Ticket Granting Server)

Is a KDC entity which grant the user tickets for the specified services.

## PAC (Privilege Attribute Certificate)

As we said Kerberos here for authentication and not authorization **BUT** he also handles a structure in every ticket that contains the user’s privileges, and it is signed with the KDC key.
This allows the AP to verify against the KDC what credentials the user has with the TGS he presents to him.

## Keys :key:

### User secret key

A key that is derived from the user password hash.

### Session key

Used during the conversation between the TGS and the User. The user gets this key inside the KRB_AS_REP packet.

### Service (AP) key

This key derivative from the NTLM hash of service owner, (which can be a user or computer account) and used for the communication between the User and the Service.
