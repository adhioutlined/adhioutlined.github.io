---
title:  "Add Plank Dock in to Ubuntu Desktop 18.04"
header:
  image: /assets/images/Workspace_182aaa.png
date:  2018-10-20
tags:
  - Linux Basic
description: ''
categories:
  - Basic
---
Just want to make some fun things with Ubuntu 18.04 desktop at my working laptop. I want add Plank dock just to be like OSX dock.

To install Plank Dock into Ubuntu 18.04 , we will installing from command line, luckily plank package was already available at Ubuntu Repository.

## Installing Plank dock

```
# apt install Plank
```
### Operating Plank
To start plank just press start button then search **PLANK** and open it, it would be appears at bottom of desktop by default.

For manage Plank Dock preference we need to press `Ctrl` + `Right click`  at plank Dock already

### Make Plank Start Automatically
To make Plank start Automatically while login, we need add autostart script for plank. Create `~/.config/autostart/plank.desktop` file with this content
```
[Desktop Entry]
Type=Application
Exec=plank
Hidden=false
NoDisplay=false
Name[en_US]=plank
Name=plank
Comment[en_US]=plank
Comment=plank
X-GNOME-Autostart-Delay=2
X-GNOME-Autostart-enabled=true
```

## Hide GNOME dock
Hide GNOME dock, so your desktop will only show Plank Dock only. Hide GNOME Dock using `gsttings`. To make it simple, I create two file.
First file to hide GNOME dock, the other one for enabled it again.

**Hide GNOME Dock**
```
gsettings set org.gnome.shell.extensions.dash-to-dock autohide false
gsettings set org.gnome.shell.extensions.dash-to-dock dock-fixed false
gsettings set org.gnome.shell.extensions.dash-to-dock intellihide false
````
make it executable with `chmod +x`, then execute it.

**Show GNOME Dock**
```
gsettings set org.gnome.shell.extensions.dash-to-dock autohide true
gsettings set org.gnome.shell.extensions.dash-to-dock dock-fixed true
gsettings set org.gnome.shell.extensions.dash-to-dock intellihide true
```
also make it executable.

![alt text](/assets/images/Workspace_1_182.png)
