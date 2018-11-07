# bake
A simple, bash-based build system
---------------------------------
This is based on the observation that many Makefiles actually call out to bash
to do the real work. That includes many tasks such as creating output directories,
checking for the existence of files, and many others that the Make language
simply doesn't allow for. This kinda defeats the purpose of having a Make language
since the moment a bash function is called directly, we lose portability to run
the Makefile on any other shell.
Moreover, the syntax and functionality of make are not that different from bash
itself, which begs the question 'why use two languages instead of one?'
Thus, 'bake' is the result of bash+make => a bash-based build system
