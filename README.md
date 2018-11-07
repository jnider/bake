# bake
## A simple, bash-based build system
This is based on the observation that many Makefiles actually call out to bash
to do the real work. That includes many tasks such as creating output directories,
checking for the existence of files, and many others that the Make language
simply doesn't allow for. This kinda defeats the purpose of having a Make language
since the moment a bash function is called directly, we lose portability to run
the Makefile on any other shell.
Moreover, the syntax and functionality of make are not that different from bash
itself, which begs the question 'why use two languages instead of one?'
Thus, 'bake' is the result of bash+make => a bash-based build system

## How To Install
'bake' will run just the same as any bash script. Personally, I like to clone
the git repository somewhere in my home directory, and then create a soft link
to another directory that contains all my scripts that is in my path. For
example:

```
~/projects$ git clone https://github.com/jnider/bake.git
~/projects$ mkdir ~/bin
~/projects$ export PATH=$PATH:~/bin
~/projects$ ln -s bake/bake ~/bin/bake
```

## How to Write A Recipe
The input to 'bake' is called a recipe. You write a recipe file to give the
instructions on how to build your particular program. This is the equivalent of
a Makefile or CMakeLists.txt file in other build systems.
Recipe files are regular bash shell scripts, but you can structure them in a
particular way to get your project up and running very quickly.
Bake will automatically look for a file called 'recipe' in the current directory.
If you want to use a different name, you can pass the -f flag like this:
```
bake -f other-file
```

### Targets
Targets are how you tell 'bake' what to build. It could be the name of an output
file, or a random name that you make up - it doesn't really matter. You can
specify the target you want to build on the command line like this:
```
bake my-target
```
If the target is dependent on other files (like in the C language, where .o
files depend on .c and .h files) you can use an array variable to specify the
dependencies in your recipe file like this:
```
main_o=(main.c header.h)
```
If you want to get really complicated, that's ok too. For example, say you are
really lazy and want to just include all .c files in particular location:
```
main_o=($(find . -name '*.c'))
```
That's the fun part of 'bake' - if you already know bash, then you can use the
commands and syntax you are already familiar with.

### Rules
Rules run when 'bake' determines something is out of date. Generally, something
is out of date if it doesn't exist, or it has a dependency that has a more
recent timestamp in the file system. A rule is just a regular bash function,
but the function name must start with the prefix "rule_". The 'bake' script will
search for all functions that start with that prefix when it is trying to run
a rule. That way, you have the freedom to write any other functions you want,
and they will not be picked up as rules.
If 'bake' determines that a rule should be run on a particular target, it will
try all rules that it can find, one by one, until it finds one that accepts the
name of that target. When you write a rule, you are responsible for the code that
tells 'bake' if the target matches or not. Generally, it looks something like this:
```
function rule_clean_roottask()
{
	local target=$1
	[ "$target" == "roottask_clean" ] || return -1

	echo "Cleaning output directory $dest_path"
	cmd="rm -rf $dest_path"
	echo $cmd
	$cmd
}
```

