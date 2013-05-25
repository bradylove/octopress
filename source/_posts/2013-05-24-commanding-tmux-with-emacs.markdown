---
layout: post
title: "Commanding Tmux With Emacs"
date: 2013-05-24 21:12
comments: true
categories: [Emacs, Tmux]
---
A few days ago I was pair programming with my good friend [Evan](http://evansparkman.com/). Evan is a [Vim](http://www.vim.org/) user, so naturally as an [Emacs](https://www.gnu.org/software/emacs/) lover I am always giving him crap about his precious Vim. However while we were pairing he showed me an awesome tool he had under his belt with Vim. Evan was using the power of [Vimux](https://github.com/benmills/vimux.git) and [tslime](https://github.com/kikijump/tslime.vim) to send commands to his Tmux session from within Vim. I was instantly jeleous and the next day I began to investigate. As it turns out Tmux makes it super simple to sand a command to any pane from any terminal. Here is what I discovered.

## The Details On Tmux

Tmux comes with the `send-keys` command that you can send along with passing in the session name, window name and pane number and the keys that you would like to send to the specified pane. Try it out, start up a new Tmux session, you can do this by giving the session a name or allowing Tmux to assign the session a number. To startup a new Tmux session with a name you can run the following.

    $ tmux new -s sample-session

Or to have Tmux assign a session number you can send `tmux new` without the `-s sample-session` argument. Then you can run `tmux ls` or `tmux list-sessions` to get a list of running Tmux sessions so that you can get the session number which is 0 in the following example.

    $ tmux new
    $ tmux ls
    0: 1 windows (created Fri May 24 20:43:18 2013) [105x129] (attached)

Now that you have your Tmux session running, go ahead and open up another terminal. From this new terminal we will send the `tmux send-keys` command, passing in the `-t` argument with the `session-name:window-name.pane-number` followed by the keys we want to send to the Tmux pane, `window-name` can also be replaced by the windows number. Here are some examples of what this would look like.

    # Executes 'ls' on pane 1 of window 1 of sample-session
    $ tmux send-keys -t sample-session:1.1 "ls" Enter

    # Executes 'bundle exec rake routes' on pane 2 of window 3 of session 0
    $ tmux send-keys -t 0:3.2

## Lets Add Some Emacs

Now that we know how to send a command to a Tmux session from another terminal its pretty easy to make this happen from Emacs. Emacs provides a function named `shell-command` that will execute any shell command you pass it. If you have read my blog post [Controlling Spotify From Within Emacs on Linux](http://bradylove.com/blog/2013/05/09/controlling-spotify-from-within-emacs-on-linux/), you may recognize this. Here is an example of how I used this command to run Cucumber in my terminal from within Emacs.

``` scheme
(defun tmux-exec-cucumber ()
  "Execute cucumber in tmux pane"
  (interactive)
  (shell-command "tmux send-keys -t 0:1.1 'bundle exec cucumber' Enter"))
```

Once this this function has been evaluated you can run it by pressing `M-x` then typing in the function name `tmux-exec-cucumber` and pressing enter. When you do it will kick off Cucumber in pane 1 of window 1 of session 0.

This is cool, howerver pretty impractical considering every time we want to send the command to a different pane we will have to edit this file, and if we add more commands, that adds more editing for different panes... What a major pain in the ass. So lets go ahead and define some variables with some defaults for the sesssion-name, window-name and pane number, then we will define a function to set these variables without having to edit any
files.


``` scheme
(setq tmux-session-name 0)
(setq tmux-window-name 1)
(setq tmux-pane-number 1)

(defun tmux-setup (x y z)
  "Setup global variables for tmux session, window, and pane"
  (interactive "sEnter tmux session name: \nsEnter tmux window name: \nsEnter tmux pane number: ")
  (setq tmux-session-name x)
  (setq tmux-window-name y)
  (setq tmux-pane-number z)
  (message "Tmux Setup, session name: %s, window name: %s, pane number: %s" tmux-session-name tmux-window-name tmux-pane-number))
```

When we run this new function with `M-x tmux-setup`, it will ask us what Tmux session, window and pane we want to send our commands too, then assigns our input to the variables that we setup. Next we need to modify our `tmux-exec-cucumber` function to use these new variables. We can do that by modifying it to look like this

``` scheme
(defun tmux-exec-cucumber ()
  "Execute cucumber in tmux pane"
  (interactive)
  (shell-command
    (format "tmux send-keys -t %s:%s.%s 'bundle exec cucumber' Enter" tmux-session-name tmux-window-name tmux-pane-number)))
```

With these changes in place when we run `M-x tmux-exec-cucumber` it will send `bundle exec cucumber` to the pane we specified when we ran `M-x tmux-setup`.

Now this is a bit better, however the function isn't very reusable so lets go ahead and change the `tmux-exec-cucumber` function to just `tmux-exec` and accept the command as a parameter and pass that command instead of `bundle exec cucumber`.

``` scheme
(defun tmux-exec (command)
  "Execute command in tmux pane"
  (interactive)
  (shell-command
    (format "tmux send-keys -t %s:%s.%s '%s' Enter" tmux-session-name tmux-window-name tmux-pane-number command)))
```

Then we will define a new function for executing the cucumber command.

``` scheme
(defun tmux-exec-cucumber ()
  "Execute 'bundle exec cucumber' in tmux pane"
  (interactive)
  (tmux-exec "bundle exec cucumber"))
```

And there you have it, `M-x tmux-exec-cucumber` will kick off cucumber in your Tmux pane still, but this function is a little easier to duplicate for adding extra commands, such as `tmux-rake-routes`.

``` scheme
(defun tmux-exec-rake-routes ()
  "Execute 'bundle exec rake routes' in tmux pane"
  (interactive)
  (tmux-exec "bundle exec rake rouets"))
```

I plan on doing some more exploration around Tmux and Emacs to see what other cool tricks I can find and to see how I can expand on this subject. Until then you can checkout my `tmux.el` file I have on my [.emacs.d Github repo](https://github.com/bradylove/.emacs.d/blob/master/init.d/tmux.el). Enjoy :)
