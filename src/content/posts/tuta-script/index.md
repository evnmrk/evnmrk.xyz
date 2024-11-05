---
title: "Bash Script to Install Tuta AppImage"
date: 2024-11-04T18:04:19-08:00
tags: ["script", "appimage", "bash"]
draft: true
---

I use Tuta as my trusted email client for over a year now, and I've noticed a bug in the Linux appimage that annoys me to no end. When Tuta needs an update, the appimage will prompt you with the choice to install the update now, but this prompt only shows up for about 3 seconds. If you have Tuta open on startup, there is a good chance you will miss this prompt.

<!--more-->

The issue is if you forget to manually install the update, ***Tuta will be uninstalled from your system.*** I do not know if Tuta is uninstalled when the prompt shows, but you will have to reinstall the appimage from Tuta's website to check your email.

## Scripts

To solve this, I created two shell scripts to automatically check if Tuta is installed, and if it is not, install it using curl. The first script, `check_tuta_exists.sh`, checks if Tuta exists, and runs the second script if it isn't. After install, it runs Tuta. The second script, `get_tutanota.sh`, is the install script, which pulls the AppImage from Tuta's webserver and gives the AppImage proper permission. I have the install logic in a separate script if I want to manually install Tuta for whatever reason.

```bash
# check_tuta_exists.sh

#!/bin/bash

if [ ! -f ~/AppImages/tutanota-desktop-linux.AppImage ] ; then
	echo "Tuta does not exist!"
	notify-send -i tutanota-desktop "Downloading Tuta..."
	bash ~/Scripts/get_tutanota.sh
	~/AppImages/tutanota-desktop-linux.AppImage -a
fi
```

```bash
# get_tutanota.sh

#!/bin/bash
curl -o ~/AppImages/tutanota-desktop-linux.AppImage https://app.tuta.com/desktop/tutanota-desktop-linux.AppImage
chmod +x ~/AppImages/tutanota-desktop-linux.AppImage
```

My AppImages are stored in the ~/AppImages/ directory. I have these scripts run on startup through Cinnamon's Startup Application application.

Feel free to use these scripts if needed.
