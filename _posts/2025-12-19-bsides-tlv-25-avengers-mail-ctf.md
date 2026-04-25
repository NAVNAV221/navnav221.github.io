---
layout: post
title: "BSidesTLV 2025: Avengers Mail CTF Writeup"
subtitle: "The challenge that left the Avengers (and the players) hanging: Mail Confusion."
categories: CTF
cover-img: /assets/img/ctf/bsidestlv25/avengersMail/avengers_banner.png
thumbnail-img: /assets/img/ctf/bsidestlv25/avengersMail/bstlv_color_no_bg.png
tags:
  [CTF, Web, BSidesTLV, Mail-Confusion, RFC2047, UTF-7, BSidesTLV2025, BSides]
comments: true
---

This weekend, I had the pleasure of contributing a Web challenge to **BSidesTLV 2025** titled **"Avengers Mail"**.

**I want to thank to [@DaCurse](https://github.com/DaCurse) for the inspiration for this challange! Thanks man :] [@DaCurse](https://github.com/DaCurse).**

The premise was simple: _"Cause if we can’t protect our Mail, you can be damn sure we’ll avenge it."_ However, it turns out that even Earth's Mightiest Heroes couldn't save this one ([Except JCTF & CamelRiders Teams!👏](https://jctf.team/writeups/BSidesTLV-2025/Avengers_Mail/))—**the challenge ended the competition with three teams that solve that challenge.**

![Challenge Statistics](/assets/img/ctf/bsidestlv25/avengersMail/statistics_challenge_bsides2025.png)

Don't feel bad, though! This challenge was designed to highlight a sophisticated and often overlooked vulnerability: **Mail Confusion via Normalization**.

Let's dive into how to solve the challenge that stumped every single player in the Web category.

# Initial Reconnaissance

When we first visit the application, we are greeted with the Avengers Mail dashboard.

![Welcome Screen](/assets/img/ctf/bsidestlv25/avengersMail/welcome_screen.png)

<p style="text-align: center;">Figure 1.0: The landing page.</p>

We see four main routes: **Home**, **Books**, **Emails**, and **Register**. As a curious attacker, the first thing we do is check the "Emails" tab, but we are immediately blocked:

![Emails Screen Unauthorized](/assets/img/ctf/bsidestlv25/avengersMail/emails_screen_unauthorized.png)

<p style="text-align: center;">Figure 1.1: The authentication wall.</p>

We need to register. But to get where we want to go, we need to know the organization's internal structure.

# Connecting the Dots

If we head over to the `/books` screen, we see a broad view of the comics store and the authors behind them.

![Books Screen](/assets/img/ctf/bsidestlv25/avengersMail/books_screen.png)

<p style="text-align: center;">Figure 1.2: The broad view of the books route.</p>

By taking a deeper look at the email signatures of the authors, we find two massive clues:

![Books Screen Emails](/assets/img/ctf/bsidestlv25/avengersMail/books_screen_emails.png)

<p style="text-align: center;">Figure 1.3: Spotting the domain discrepancies.</p>

1.  **`stan_lee@comics.staff`**: This follows the "correct" internal domain naming convention.
2.  **`thor@staff.comics`**: This domain is flipped. This is a subtle hint that the system's domain validation might be susceptible to bypasses or confusion.

# Registration and the "Attacker" Reveal

The registration screen tells us that it's for "authorized personnel only" and expects a `@comics.staff` address.

![Register Screen](/assets/img/ctf/bsidestlv25/avengersMail/register_screen.png)

<p style="text-align: center;">Figure 1.4: The registration gate.</p>

Let's try to register as a legitimate-looking user: `foo@comics.staff`.

![Register Foo Success](/assets/img/ctf/bsidestlv25/avengersMail/register_screen_foo_email_successfully.png)

<p style="text-align: center;">Figure 1.5: Successfully joining the organization.</p>

Now that we are logged in, we revisit the **Emails** path. This is the first time we see our own inbox, but more importantly, we see the external email address of the target/attacker:

![Emails Screen](/assets/img/ctf/bsidestlv25/avengersMail/emails_screen.png)

<p style="text-align: center;">Figure 1.6: Revealing the attacker address.</p>

Target address: `attacker@bsides-m2nlntqzmjq.attacker-server.com`

# The Wall: Domain Validation

If we try to register directly with the attacker's email, the system rejects us:
`"Email must belong to an allowed domain"`

The application is strictly checking for the `@comics.staff` suffix. To bypass this, we need to explore how the **Internet Message Format** handles different representations of an address.

### Beyond simple addresses

Most people think an email is just `user@domain.com`, but the RFCs allow much more:

- **Encoded Display Name (RFC 2047)**: `=?UTF-8?B?Sm9zw6lwaGluZQ==?= <user@example.com>`
- **Quoted-string with escaped characters**: `"user\"name\test"@example.com` (Allows reserved characters inside quotes).

In this challenge, we leverage the fact that many mail parsers will **decode** encoded headers before processing them, while the **validator** might only look at the raw string.

# The Vulnerability: Mail Confusion

The core of the challenge is achieving **Mail Confusion**. By using **UTF-7** encoding within an RFC 2047 "encoded-word", we can hide the external domain from the validator while ensuring the parser resolves it to the attacker's email.

UTF-7 is a 7-bit encoding that can represent characters like `@` using sequences like `&AEA-`.

### Building the Payload

We need a payload that passes the domain check by ending in `@comics.staff`, but contains our target email in the local part.

- `@` → `&AEA-`
- `<space>` → `&ACA-`

**The Payload Structure:**
`=?utf-7?q?attacker&AEA-YOUR-SERVER.com&ACA-?=@comics.staff`

The complexity here—and the reason for the 0% solve rate—is the precise construction of this RFC 2047 header. If any part of the `=?utf-7?q?...?=` syntax is slightly off, the parser fails, and the validation blocks the "illegal" characters.

# Victory: The Flag

We submit our crafted UTF-7 payload in the registration field:

![Bypass Domain Registration](/assets/img/ctf/bsidestlv25/avengersMail/bypass_email_domain_restriction_with_attacker_mail.png)

<p style="text-align: center;">Figure 2.0: Submitting the encoded payload.</p>

The validator sees the `@comics.staff` at the end and gives the green light. However, the backend parser decodes the UTF-7 string, identifies the external attacker server, and routes the internal flag email to our inbox!

![Flag Found](/assets/img/ctf/bsidestlv25/avengersMail/bypass_email_domain_restriction_with_attacker_mail_flag.png)

<p style="text-align: center;">Figure 2.1: The Avengers have been avenged.</p>

# Final Words

This was a "rabbit-hole" challenge by design. It forces you to look past the surface of a simple string and understand how legacy RFCs interact with modern web frameworks.

Even though nobody solved it during the competition, the concept of **Normalization Confusion** is a vital one for any modern bug hunter to have in their arsenal.

If you have any questions or want to dive deeper into mail exploits, I'm always available on Discord or X!

**Thanks to everyone who gave it their best shot at BSidesTLV 2025!**

— Nave
