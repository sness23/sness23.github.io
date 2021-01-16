# jan1421 23:05

<iframe width="560" height="315" src="https://www.youtube.com/embed/p5NX1FC-7-w" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<iframe width="560" height="315" src="https://www.youtube.com/embed/6bTx2-ezsyE" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


# emacs lisp

i have an insert-date function
that i bind to `M-i`, i found
it somewhere on the web

```
(defun insert-date ()
  "Insert date at the current cursor position in the current buffer."
  (interactive)
  (insert  
   (format-time-string "# %b%d%y %H:%M%n")
   ))
```

format-time-string has a ton of options, but
a three-letter month that is lower-case does not
appear to be one yet.

to do this, i wrapped the existing `format-time-string` function
with the downcase function in emacs lisp

```
(defun insert-date ()
  "Insert date at the current cursor position in the current buffer."
  (interactive)
  (insert  
   (downcase (format-time-string "# %b%d%y %H:%M%n"))
   ))
```

this is one of the real benefits to having a system
that is written in a language that you can use

even better though would be if emacs had an API and
i could use whatever language i wanted to control it

same thing with pymol

# ubuntu alacritty copy paste with keyboard

[Okay, so how to paste text in the Linux Terminal?](https://www.fosslinux.com/14088/how-to-copy-and-paste-commands-in-linux-terminal.htm)

> Due to the above-discussed issue, the modern Terminal developers applied Ctrl + Shift + V for paste. Similarly Ctrl + Shift + C for copy function. Alternatively, you can right-click and select paste from the menu for pasting the copied command-line.

# the forest

```
walking in the forest
all alone
```

<iframe width="560" height="315" src="https://www.youtube.com/embed/r54jwZvtJck" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# fun

```
why cant we just play music
and make art
even if we aren't the best
in the whole wide world?
it's fun to play music
to make art
to walk in the forest

we should get to have fun
if we want to have fun

i personally want to have fun
is that so wrong?
```

# the dao

the way that can be spoken of
is not the true dao
