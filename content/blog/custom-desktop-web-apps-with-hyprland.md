+++
title = 'Custom Desktop Web Apps with Hyprland'
date = 2026-02-17

[taxonomies]
tags = [
  "Arch Linux",
  "Hyprland",
  "Desktop entry",
  "bash",
  "jq",
  "Json",
  "Chromium",
  "Vimium",
  "vim"
]

[extra]
toc = true
+++

The reason I use Linux is because I like to configure my system to be exactly how I want it.

This is why I went with [Arch Linux](https://archlinux.org/) *(btw)*. Arch may not be for everyone but when you are picky like me when it comes to what software goes on your machine, it's simply the perfect choice.

On top of that, I use [Hyprland](https://hypr.land/) as my [tiling window manager](https://en.wikipedia.org/wiki/Tiling_window_manager) *(it's an hybrid window manager to be more accurate, but you get the idea)*.

While I recognize the mouse to be [a truly magnificent invention](https://www.youtube.com/watch?v=yJDv-zdhzMY), the less I have to use it, the better I feel.
I am a keyboard maximalist, I don't want my hands to ever leave the keyboard so a tiling window manager fits my workflow the best.

# Predefined Workspaces

The way I setup my environment is that I have ten workspaces and each of them gets assigned specific applications:

1. [FreeTube](https://freetubeapp.io/)
2. Web browsers ([Librewolf](https://librewolf.net/) / [Chromium](https://github.com/ungoogled-software/ungoogled-chromium), [qutebrowser](https://qutebrowser.org/))
3. Terminal ([kitty](https://sw.kovidgoyal.net/kitty/) / [ghostty](https://ghostty.org/))
4. Email client
5. Chat apps (Discord, Signal, Teams, etc.)
6. Office ([OnlyOffice](https://www.onlyoffice.com/) / [LibreOffice](https://www.libreoffice.org/))
7. Video games & graphics ([Gimp](https://www.gimp.org/))
8. Remote desktop ([Remmina](https://remmina.org/)) & Virtual Machines
9. Misc & Tools ([qBittorent](https://www.qbittorrent.org/), Spotify, etc.)
10. Random stuff *(I usually bind it to my second monitor)*

The great thing about this is that I don't have to think about where my windows are. Switching to a specific application is built into my muscle memory.
If I want to search on the web, I press <kbd>SUPER+2</kbd>. Need to read an email? <kbd>SUPER+4</kbd>, etc.
It's fast, there is no mental overhead, I love it.

When you spend 8+ hours a day on a computer, the mental fatigue of having to <kbd>Alt+Tab</kbd> and manually search through your windows really builds up.

The Workspace config is available on [GitHub](https://github.com/coko7/.dotfiles/blob/main/.config/hypr/window_rules_workspaces.conf).

# Creating custom Desktop Web apps

OK, now you have a basic idea of what my workflow looks like. So why do I need desktop web apps?
And by that, I mean: Web apps that I can launch like any other application.

Well, as you know, I like to bind applications to workspaces. But sometimes, I have to use applications or services that are not available on desktop.
I could simply stick to using them in my web browser, and, that's what I do for some of them. But, when I find myself using a web app a lot, I want to have it available on my desktop directly.

To create the desktop web app, we need three things:

- a desktop entry
- an executable script
- some Hyprland rules

In this article, I will walk you through the process of creating a desktop web app for Outlook.

## Desktop entry

First, we need to create a custom [desktop entry](https://wiki.archlinux.org/title/Desktop_entries) (`*.desktop` files):

```desktop,linenos,hl_lines=7
[Desktop Entry]
Version=1.0
Name=Outlook
Comment=Read your emails
GenericName=Work Email Client
Keywords=Work;Email;Outlook
Exec=/home/coko/.config/local/bin-sh/work/chromium-outlook.sh
Icon=outlook
Terminal=false
Type=Application
StartupNotify=true
Categories=Network;Email;
```

The above snippet is a desktop file that I placed in my `~/.config/local/share/applications/` directory.

When I press <kbd>SUPER+R</kbd>, it starts my [application launcher](https://wiki.archlinux.org/title/Category:Application_launchers) which automatically picks up this entry and makes it available to me.
I use [rofi](https://github.com/davatorium/rofi) as my application launcher but if you use a different one, it should behave similarly.

You can specify a lot of things in a desktop entry, but what we want to focus on is the following:

```bash
Exec=/home/coko/.config/local/bin-sh/work/chromium-outlook.sh
```

The `Exec` value is the action that will be triggered when you launch the application. In this case, it is a custom bash script named `chromium-outlook.sh`.

Never heard of bash? [Learn it in Y minutes!](https://learnxinyminutes.com/bash/)

## Executable script

Now, let's look into the bash script from the desktop entry:

```bash,linenos
#!/usr/bin/env bash

DATA_DIR="$XDG_CONFIG_HOME/chromium-work-outlook"
chromium-nerd-app.sh "https://outlook.office.com" "$DATA_DIR"
```

It's tiny, only two lines without the [Shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)).
First, I define the data directory for [Chromium](https://github.com/ungoogled-software/ungoogled-chromium), which is the web browser I use for my custom desktop web apps.
This is mostly for isolation purposes, the actual line we want to have a look at is **line 4**:

```bash
chromium-nerd-app.sh "https://outlook.office.com" "$DATA_DIR"
```

It calls another script ([`chromium-nerd-app.sh`](https://github.com/coko7/.dotfiles/blob/main/.config/local/bin-sh/global/chromium-nerd-app.sh)) with two arguments (a URL and the data dir we defined).
Let's checkout this new bash script:

```bash,linenos
#!/usr/bin/env bash

APP_URL="$1"
[[ -z "$APP_URL" ]] && echo "APP_NAME required!" && exit 1

DATA_DIR="$2"
[[ -z "$DATA_DIR" ]] && echo "DATA_DIR required!" && exit 1

VIMIUM_EXT_ID='dbepggeogbaibhgnhhndojpepiihcmeb'
VIMIUM_VERSION='2.3.1_0'
CHROMIUM_EXT_PATH="$XDG_CONFIG_HOME/chromium/Default/Extensions"
VIMIUM_EXT_PATH="$CHROMIUM_EXT_PATH/$VIMIUM_EXT_ID/$VIMIUM_VERSION"

/usr/bin/chromium \
    --app="$APP_URL" \
    --disable-features=WaylandWpColorManagerV1 \
    --user-data-dir="$DATA_DIR" \
    --load-extension="$VIMIUM_EXT_PATH"
```

Okay, this one looks a little more daunting, but it's not that complicated actually.

The first few lines simply ensure that the script is called with two arguments (the app URL and the data directory).

Let's skip ahead to **lines 9-12**:

```bash
VIMIUM_EXT_ID='dbepggeogbaibhgnhhndojpepiihcmeb'
VIMIUM_VERSION='2.3.1_0'
CHROMIUM_EXT_PATH="$XDG_CONFIG_HOME/chromium/Default/Extensions"
VIMIUM_EXT_PATH="$CHROMIUM_EXT_PATH/$VIMIUM_EXT_ID/$VIMIUM_VERSION"
```

This part is optional ***but it's my favorite part***.
If you are familiar with [Vimium](https://vimium.github.io/), you already know what's coming.
But, basically: Vimium is a browser extension that brings vim keybindings in your web browser.
Some think it's overkill, I cannot live without it.

These four lines define the path to the Vimium extension on my system.
The same could have been achieved with a single line but this is more readable.

Then, finally, the heart of the script *(where the magic happens ✨)*:

```bash,linenos
/usr/bin/chromium \
    --app="$APP_URL" \
    --disable-features=WaylandWpColorManagerV1 \
    --user-data-dir="$DATA_DIR" \
    --load-extension="$VIMIUM_EXT_PATH"
```

Let's break it down line by line:

1. Launch the Chromium binary: `/usr/bin/chromium`
2. `--app=<URL>` tells Chromium to start in a **application** mode, it will hide menu bars, disable extensions, etc.
3. `--disable-features=WaylandWpColorManagerV1` is for fixing some color issues on Wayland (to be honest, I don't remember why it's there but there must be a reason)
4. `--user-data-dir="$DATA_DIR"` tells Chromium to use the data directory passed as second argument `$2`
5. `--load-extension="$VIMIUM_EXT_PATH"` instructs Chromium to specifically load the Vimium extension

And that's it!

With this, whenever I launch Outlook from my app launcher, it will open a custom instance of Chromium with Outlook opened. It looks and feels like
a desktop app but it is actually a disguised Chromium browser.

We are almost done, but there is still one important thing left: **binding the app to a Hyprland workspace**.

## Hyprland rules

If you remember my workspace setup, I want my email client to open on workspace 4.
To do this, I need to define some [window rules](https://wiki.hypr.land/Configuring/Window-Rules/).

Since my config is quite big, I have decided to split it into multiple files and I keep workspace-related window rules in [`window_rules_workspace.conf`](https://github.com/coko7/.dotfiles/blob/main/.config/hypr/window_rules_workspaces.conf):

```ini,linenos
windowrule = workspace 4, match:class ^(chrome-outlook.office.com__-Default)$
windowrule = match:class ^(chrome-outlook.office.com__-Default)$, tile on
```

Hyprland configuration is rather easy to understand even if it's your first time seeing it:

1. Assign Outlook to **workspace 4**
2. Force the Outlook window to be tiled instead of floating

In this case, the `^(chrome-outlook.office.com__-Default)$` [regex](https://en.wikipedia.org/wiki/Regular_expression) matches the Outlook window.
Okay, sure... But where did I find weird-looking class name?

That's Hyprland's strongest suite in my opinion, it comes with a bunch of tools that expose metadata about desktop objects (windows, monitors, devices, etc.).
Here, the command I use to get the class name for Outlook is the following:

```bash
hyprctl clients -j | jq '.[].class'
```

**More nonsense commands 😡 !!??**

Calm down, I will explain everything:

- `hyprctl clients -j`:
  - `hyprctl` is the main Hyprland command line tool to query desktop objects
  - `clients` is a subcommand of `hyprctl` that uses the Hyprland API to find the list of clients (opened windows/applications)
  - `-j` tells Hyprland to format the output as [JSON](https://en.wikipedia.org/wiki/JSON) *(that's why I love Hyprland)*
- `jq '.[].class'` relies on [jq](https://github.com/jqlang/jq) to process the JSON output from Hyprland and format it to only show individual class names.

When I run this on my system, this is what I get:

```console,linenos
$ hyprctl clients -j | jq '.[].class'
"librewolf"
"chrome-outlook.office.com__-Default"
"kitty"
```

Spot anything familiar? I don't know about you but the second class `chrome-outlook.office.com__-Default` sure sounds like it could be our Outlook window.

And that's it! Now when I launch Outlook, it will always be opened on workspace 4. Easy enough, right?

# Conclusion

It may seem like I have to do a lot of work to turn a simple website into a desktop application. But, in reality, all I have to do is create the desktop entry and the Hyprland rule.
And creating these takes one or two minutes.
Most of the heavy lifting is performed by the `chromium-nerd-app.sh` script.

I hope you liked this article and learned some neat things. I had a lot of fun writing it!
All the configuration and scripts shown here can be found on my [GitHub](https://github.com/coko7/.dotfiles):

- [Hyprland files](https://github.com/coko7/.dotfiles/tree/main/.config/hypr)
- [chromium-nerd-app.sh script](https://github.com/coko7/.dotfiles/blob/main/.config/local/bin-sh/global/chromium-nerd-app.sh)

If you have any question or feedback, feel free to email me at [contact@coko7.fr](mailto:contact@coko7.fr). Please consider sharing this around if you found it helpful *(or even if it was not helpful)*.

Alright, I am done with my rambling. Bye bye 👋

`:wqa`
