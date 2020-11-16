---
layout: post
title: kaniyeri/efficient-screenshot
tags: [utility]
comments: true
---

This is a extended readme on how to use kaniyeri/efficient-screenshot.

I created efficient-screenshot because there was no good method of taking screenshots and automatically arranging them into folders for easy categorisation. The current method of taking screenshots involve the snipping tool which opens into an editor and a file explorer to select directory... complete waste of time especially if you want to take multiple screenshots in lectures or chats.

efficient-screenshot was made for this.

# Installation

Please make sure you have the current version of Python installed. Please find the packages [here.](https://www.python.org/downloads/)

This version of efficient-screenshot requires packages which are not bundled with python. The extra packages are listed in prerequisites.txt. The packages can be installed automatically by running setup.py(also bundled with your clone of the repository.)

In case any other packages are required for running the file, it will be updated in the prerequisites.txt and can be refreshed by running setup.py again. (setup.py pulls package names from prerequisites.txt)

---
# Using the utility

To use efficient-screenshot, run basic.py. It will open a command window displaying information and the current time. The time shown is the time of opening the file. It does not update.

The date and time is used in naming the folder and file. This is by design, it makes sure there are no clashes in file names *ever*.

Press LCtrl to take screenshots. A new folder is created with the current date and time and the screenshot is saved in the folder. All subsequent screenshots are named sequentially inside the folder.

To create a new folder, close and relaunch the application. This is useful if you want to split into categories.

---

# Use cases

I created multiple folders correlated to courses I have in my first year. I have folders named EE113, MA 109, etc. I have copied basic.py into all of these folders. Every lecture I open basic.py and keep it in the background and click LCtrl whenever I see something useful.

The screenshots get saved into a folder with date and time for future reference.

Although people who have used this utility also mentioned using it for saving chats from desktop chat programs.

--- 

# Downloads
Clone or download from [https://github.com/kaniyeri/efficient-screenshot](https://github.com/kaniyeri/efficient-screenshot)
