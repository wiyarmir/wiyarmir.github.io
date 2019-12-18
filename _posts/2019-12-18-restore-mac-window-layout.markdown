---------
layout: post
title: "Maintaining and restoring window arrangements in MacOS"
date: 2019-12-18 13:37:00 +01:00
author: wiyarmir
comments: true
---------



# Install Homebrew

Go to https://brew.sh/ and follow instructions

# Install displayplacer

Instructions from the GitHub repo are

```
brew tap jakehilborn/jakehilborn && brew install displayplacer
```

# Copy your layout configuration

After installing the tool, execute `displayplacer list` in a terminal. Copy the output, that will look something like:

```
displayplacer "id:0BF37642-C36D-6E90-01B5-9C8D747DA7F5 res:2560x1440 scaling:off origin:(0,0) degree:0" "id:E8102887-70FC-9A98-8839-157ACFB40B52 res:1440x2560 scaling:off origin:(-1440,-404) degree:90"
```

# Create a Quick Action in Automator

Open Automator, and create a new Quick Action. If you can't find Automator, you can do `Cmd+Space` and type its name.

## Add a `Run Shell Script` action

![](https://user-images.githubusercontent.com/172084/71083415-47fea000-2193-11ea-9461-b8b5e7b948e9.png)

## Paste the output of the layout configuration step

Put an absolute path to `displayplacer` if you have issues with the shell path (if it complains about not finding the displayplacer executable)

![](https://user-images.githubusercontent.com/172084/71083582-ac216400-2193-11ea-986e-13434bea8109.png)

If you need to know the full path to `displayplacer`, execute `which displayplacer`.

## Save the Quick Action and give it a memorable name 

Memorable as in you will have to find it later in a different place.

![](https://user-images.githubusercontent.com/172084/71083725-f9053a80-2193-11ea-89d4-337be7bd42e4.png)

# Create a shortcut

## Open Shortcut settings

It's in System Preferences > Keyboard > Shortcuts

## Find your saved Quick Action

It usually is right at the end of the list, so scroll away.

![](https://user-images.githubusercontent.com/172084/71083839-41245d00-2194-11ea-811e-2ae9e9fdc211.png)

## Assign a key binding to it

Make sure it's not one that is already defined in the system (you can disable it if it is) or one that you would use regularly on your day-to-day. 

I've used `Cmd+Alt+Shift+R` but your mileage may vary.

# Happy rotating

Try it out next time your screens go bonkers and if it saved you any time say thanks!
