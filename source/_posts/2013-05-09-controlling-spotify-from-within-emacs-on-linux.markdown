---
layout: post
title: "Controlling Spotify From Within Emacs on Linux"
date: 2013-05-09 20:13
comments: true
categories: [Emacs, Spotify, Linux]
---
A while back I switched from VIM to Emacs and I have been meaning to write some blog posts on some cool things I have been able to do within Emacs, so here is one on how I control spotify from within Emacs.

## The Problem

So recently I built a new computer, I was debating between getting a [System76](https://www.system76.com/) laptop, a new Apple MacBook Pro or building a desktop and running Linux on it. I ended up going for building a desktop and running Linux on it. Originally I started out with [Linux Mint 14](http://linuxmint.com/), then I switched to [Ubuntu](http://www.ubuntu.com), then I wanted something more powerful and switched to [ArchLinux](http://www.archlinux.org). A common problem I had across all three of these distributions was out of the box, I could not use the media keys on my keyboard to control [Spotify](http://www.spotify.com). They would work in with other media players such as [Rhythmbox](http://projects.gnome.org/rhythmbox/) or [Audacious](http://audacious-media-player.org/) but not Spotify.

## The Solution

99% of the time that I am listening to music on this computer I have Emacs open, so I decided to fix this in Emacs rather than solving this problem through the system where a solution may not work from one desktop environment to the next.

What I did to fix this problem was described an Elisp function to send a command to spotify via `dbus-send`, then created functions for toggling play/pause, previous, and next. Then I created some key bindings for those functions. Here is that code on how I did that.

``` elisp spotify.el
(defun spotify-linux-command (command-name) "Execute command for Spotify" (interactive)
  (setq command-text (format "%s%s" "dbus-send --print-reply --dest=org.mpris.MediaPlayer2.spotify /org/mpris/MediaPlayer2 org.mpris.MediaPlayer2.Player." command-name))
  (shell-command command-text))

(defun spotify-toggle () "Play/Pause Spotify" (interactive)
  (spotify-linux-command "PlayPause"))

(defun spotify-previous () "Starts the song over in Spotify" (interactive)
  (spotify-linux-command "Previous"))

(defun spotify-next () "Next song in Spotify" (interactive)
  (spotify-linux-command "Next"))

(global-set-key (kbd "<f7>") 'spotify-previous)
(global-set-key (kbd "<f8>") 'spotify-toggle)
(global-set-key (kbd "<f9>") 'spotify-next)
```
