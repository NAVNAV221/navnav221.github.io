---
permalink: "/htb/"
layout: page
title: Hack The Box writeups
subtitle: Conclusions from the HTB machines and CTF challenges I've solved.
share-title: "Hack The Box & CTF Writeups"
share-description: "Hack The Box machine writeups and CTF challenge solutions by Nave Ben Naim (navnav221) - Active Directory, Kerberos, and web exploitation."
---

These are writeups of Hack The Box machines and CTF challenges I've worked
through. I try to write the *conclusions* — the technique and why it worked —
rather than a command-by-command walkthrough.

<ul>
{% for post in site.posts %}
  {% if post.tags contains "HTB" or post.tags contains "CTF" or post.categories contains "CTF" %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span class="text-muted"> &mdash; {{ post.date | date: site.date_format }}</span>
    {% if post.subtitle %}<br><small class="text-muted">{{ post.subtitle }}</small>{% endif %}
  </li>
  {% endif %}
{% endfor %}
</ul>

Everything else lives on the [blog index](/) or under
[categories](/categories/).

&mdash; [Nave Ben Naim](/aboutme/)
