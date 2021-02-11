---
title:  "Defending plaintext email against URL Defense<br> (and Safelinks)"
date:   2021-02-10 08:00:00 -0500
layout: singlemath
categories: Code Tools

---

[Get urldec.py](https://gist.github.com/OVlasiuk/7afbbe4fc75e27ed408a332a2b3f2494)

Over the last couple of years, an increasing number of universities in the US
(e.g. [1],[2],[3]) have introduced a link rewriting system to their email
services, the commonly used ones being [*URL
Defense*](https://www.proofpoint.com/us/resources/data-sheets/essentials-url-defense)
and
[*Safelinks*](https://support.microsoft.com/en-us/office/advanced-outlook-com-security-for-office-365-subscribers-882d2243-eab9-4545-a58a-b36fee4a46e2)
from *Outlook* (URLD and SL). Such tools are hailed as *the* solution to scam and phishing
emails, even though there appears to be no supporting experimental data that would be publicly available. Instead, it is assumed that the university employees
are too busy to understand the links even in their own emails, and should pay a
corporation to do it for them. One effect of this, witnessed personally, is that
some users will be comfortable embedding in their professional
correspondence links redirecting to third-party trackers — which, however "not malicious"
they might be for URLD/SL, still have *no* business snooping on other university users.
All this because a link is no longer a transparent address, but a cryptic object, to
be understood only by the "experts".

Regardless of the efficacy of link rewriting, one thing is certain: it makes a
mess out of plaintext email. And if that's the kind of email you like
reading — good old text in the terminal — you will find that a harmless link in
your colleague's signature will explode to several insanity-inducing lines in
`mutt` (or your favorite plaintext MUA) after being rewritten first by URLD, and then SL.

Enter `display_filter`. From `neomutt`'s manual:
```
display_filter
Type: command
Default: (empty)

When set, specifies a command used to filter messages. When a message is viewed
it is passed as standard input to $display_filter, and the filtered message is
read from the standard output. 
```
That is, we should be able to pipe our emails through an external utility that
will filter out rewritten links and fix them.

Fortunately, *Proofpoint* (the company behind URLD) have kindly
[provided](https://help.proofpoint.com/Threat_Insight_Dashboard/Concepts/How_do_I_decode_a_rewritten_URL%3F)
a Python script to decode their links. The version of the script on their
website contains a bug which causes it to break on URLs with non-Latin
characters, such as this one:

`https://uk.wikipedia.org/wiki/Остроградський_Михайло_Васильович`

After fixing this bug and adding the ability to rewrite *Safelinks* URLs, we
have a utility that restores the jumbled mess of a rewritten link to its original
glory. To fix links, we pipe them through the [script](https://gist.github.com/OVlasiuk/7afbbe4fc75e27ed408a332a2b3f2494) and collect `stdout`.

<br>

For incoming mail, store `urldec.py` somewhere, make executable, and then add 
```
set display_filter="~/script/folder/urldec.py"
```
to your `muttrc`. This still will not fix the outgoing emails, as
`display_filter` is used only in the pager, and not for response. To rectify that,
we can use `vim`'s ability to pipe buffers through external commands, and add
```
set editor = "vim -c '%!~/script/folder/urldec.py'"
```
to `muttrc`. This way, `vim` will pipe the message through `urldec.py` every
time it is invoked from `mutt`.


<br>


Conclusion:
- Running the script every time an email is opened is still very fast. In fact,
  rendering html emails in `w3m` is likely the slowest step in opening them.
- Even though using `mutt` to read email in the terminal turns you into an invincible hacker,
  it is wise not to follow obscure links, in terminal or elsewhere.




    [1]: https://faq.its.fsu.edu/email/spam-and-junk-email-protection/proofpoint-targeted-attack-protection/why-am-i-seeing
    [2]: https://blink.ucsd.edu/technology/email/security/url-defense.html
    [3]: https://support.uidaho.edu/TDClient/40/Portal/KB/ArticleDet?ID=1326
