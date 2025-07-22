---
layout: post
author: Parham
title: Zotero Links Not Working in Obsidian? It Might Be Snap
---

In this blog post, I explain how I managed to connect Zotero to Obsidian on Ubuntu 22.04. I am trying to officially get into the academic research world and it is the first time that I am supposed to do literature review. Knowing that things could get too complicated too quickly, I decided to seek a few tools that could help me organize the literature I’m reading. A quick search in YouTube led me to Obsidian and during the search I figured out about the Zotero integration community plugin it has. Getting started with this `integration` had a few challenges for me that I discuss in the rest of this post.

I’m using Ubuntu 22.04 and had Zotero previously installed. I also installed Obsidian using snap in the beginning. To set up both Zotero and Obsidian you need to install [Better BibTeX (BBT)](https://retorque.re/zotero-better-bibtex/) plugin for Zotero and also [Zotero Integration](https://github.com/mgmeyers/obsidian-zotero-integration) for Obsidian. I did both of these following this awesome video:

<iframe style="display: block; margin: 1rem auto; width: 600px; height: 400px;" src="https://www.youtube.com/embed/zEYp0BJL7MU?si=SvMYPnAAFhngCvZA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

After setting everything up and importing my highlights from Zotero, I noticed that the links to Zotero items weren’t working; by that I mean clicking them from within Obsidian did nothing. To resolve the issue I did a couple of steps which might help anyone who is struggling to get this going.

First, I tried to find if there were errors relating this issue, since I did not get any warnings.

```bash
~ journalctl --since "5 minutes ago" | grep obsidian
~ journalctl --since "5 minutes ago" | grep zotero
```

the response was always from Obsidian and nothing from Zotero, so I gave up on Zotero errors:

```bash
Jul 21 12:46:05 your-host obsidian_obsidian.desktop[8139]: Opening URL: zotero://open-pdf/library/items/8DNCPSUA?page=9&annotation=PEYX9TKS
Jul 21 12:46:05 your-host obsidian_obsidian.desktop[57602]: gio: zotero://open-pdf/library/items/8DNCPSUA?page=9&annotation=PEYX9TKS: The specified location is not supported
```

you can recognize the error:

```bash
gio: zotero://open-pdf/library/items/8DNCPSUA?page=9&annotation=PEYX9TKS: The specified location is not supported
```

you can replicate this error like the following:

```bash
~ xdg-open dfsdsf://salam
gio: dfsdsf://salam: The specified location is not supported
```

which indicates one thing; My computer doesn’t understand the `zotero://` protocol. There could be a few root causes:

1. I have not told my computer what `zotero://` means. or
2. I’ve defined it, but Obsidian has no access to the definitions.

To make sure it is not the first reason, I had to register a handler for Zotero mime type. To do that I navigated to ~/.local/share/applications where you usually have your `.desktop` files. I could even see my `zotero.desktop` there which looked like this:

```bash
[Desktop Entry]
Name=Zotero
Exec=bash -c "$(dirname $(realpath $(echo %k | sed -e 's/^file:\\/\\///')))/zotero -url %U"
Icon=/path/to/Zotero_linux-x86_64/icons/icon128.png
Type=Application
Terminal=false
Categories=Office;
MimeType=text/plain;x-scheme-handler/zotero;application/x-research-info-systems;text/x-research-info-systems;text/ris;application/x-endnote-refer;application/x-inst-for-Scientific-info;application/mods+xml;application/rdf+xml;application/x-bibtex;text/x-bibtex;application/marc;application/vnd.citationstyles.style+xml
X-GNOME-SingleWindow=true
```

which as you can see, completely accounts for `zotero://` by mentioning x-scheme-handler/zotero as a [MimeType](https://specifications.freedesktop.org/desktop-entry-spec/0.9.5/mime-types.html), but I did not know that it is enough at the time. Here’s what I did:

```bash
~ touch zotero-uri-handler.desktop

~ cat <<EOF > zotero-uri-handler.desktop
[Desktop Entry]
Name=Zotero URI Handler
Exec=/path/to/Zotero_linux-x86_64/zotero -uri %u
Type=Application
NoDisplay=true
MimeType=x-scheme-handler/zotero;
EOF

~ update-desktop-database ~/.local/share/applications
```

which defines a new `.desktop` and then adds it to the database, which means that my system knows that it could use `Zotero URI Handler` to handle `x-scheme-handler/zotero` . Clicking on Obsidian links still did not do anything.

I tried it another way; Now that I had the right handler in place, I tried to declare it as the main handler for `x-scheme-handler/zotero` :

```bash
~ xdg-mime default zotero-uri-handler.desktop x-scheme-handler/zotero
```

which could be verified with:

```bash
~ xdg-mime query default x-scheme-handler/zotero
zotero-uri-handler.desktop
```

This proves that my computer knows what `zotero://` means. To verify even further I tried:

```bash
~ xdg-open "zotero://open-pdf/library/items/8DNCPSUA?page=9&annotation=PEYX9TKS"
```

and it opened up Zotero and navigated to the exact file, page and annotation. We’ve seen this link show up in `journalctl` before. It means that Obsidian is trying to open the right link, and under right circumstances, that link actually does what we want. But, still when we click on the link in Obsidian, it doesn’t work. So, we have the right to think that maybe Obsidian doesn’t see what our computer sees about the Zotero mime type.

Referring to [this quote](https://ubuntu.com/blog/a-guide-to-snap-permissions-and-interfaces) about snap packages:

> To support this, each package is *sandboxed* so that it runs in a constrained environment, isolated from the rest of the system…
> 

I would say my problem has always been the case that Obsidian has no way of knowing what `zotero://` is and even if it did, it didn’t have access to my Zotero since it is running in a sandbox. In the final step, I uninstalled Obsidian from snap and installed the .deb package and it solved my problem. Finally I was able to navigate to the correct page and annotation of articles using links in Obsidian.

TL;DR: If Zotero links don’t work in Obsidian on Ubuntu, and you installed Obsidian via Snap, try the `.deb` version instead; The snap sandbox might be blocking url handlers.

Thank you for reading this post. I’d appreciate any recommendations or feedback.