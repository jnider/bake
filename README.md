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

Make also tries to be useful by referring to a lot of built-in rules that try
to guess what I'm doing based on the most common programming practices.
Unfortunately, many of these common practices are not really that common
(when was the last time you actually write a yacc parser script?) and it takes
time for make to run through all of these rules which are (the majority of the
time) useless. That slows down the build, leading people to write other tools
to avoid all this overhead.

## How To Install
'bake' will run just the same as any bash script. Personally, I like to clone
the git repository somewhere in my home directory, and then create a soft link
to another directory that contains all my scripts that is in my path. For
example:

```
~/projects$ git clone https://github.com/jnider/bake.git
~/projects$ mkdir ~/bin
~/projects$ export PATH=$PATH:~/bin
~/projects$ ln -s $(realpath bake/bake) ~/bin/bake
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
That's the fun part of 'bake' - if you already know bash, then you can use the
commands and syntax you are already familiar with.
If you don't provide a target, 'bake' will try to find one called 'main'. If
you want to be able to run 'bake' from the command line without any other
parameters, add a rule to your recipe that sets your output file as a
dependency of 'main'. You can see the section below on Dependencies for more
details on how to do this.

### Input Files
Often there will be a list of input files that you wish to perform actions on
such as compiling, assembling, etc. These lists can be generated in many ways
ranging from a completely manual process (where each filename is written
explicitly) to a complicated set of automatic functions that find files based
on search criteria. No matter how the list is generated, it should be stored
in a Bash array. The following example specifies a list of .c files:
```
c_src=(main.c usb.c loader.c video.c)
```
Another more automated method may find all of the .c files in the current
directory:
```
c_src=$(ls *.c)
```
A 3rd example finds all of the .c files in all of the subdirectories:
```
c_src=$(find . -name '*.c')
```
This list can be processed further to generate a list of targets.

### Dependencies
If the target is dependent on other files (like in the C language, where .o
files depend on .c and .h files) you can use an array variable to specify the
dependencies in your recipe file. The name of the array is important - Bake
will look for a variable that matches the target name in order to know its
dependencies. But there is a small snag; it is not legal syntax to use
certain characters in Bash variable names that are legal to use in filenames.
The most trivial example is the '.' character. This is a very common character
to use in filenames, but it cannot be used in a Bash variable name. Instead,
Bash (and therefore recipe writers as well) must replace these characters with
and underscore '_'. Thus for a target `main.o`, its dependencies will be
found in the array variable `main_o`. There are 3 such characters:
* _._ (period)
* _/_ (forward slash)
* _-_ (hyphen)

[See function filter_name() for more details][1]

Just like writing lists of input files, dependencies are Bash arrays:
```
main_o=(main.c header.h)
```
As mentioned earlier, 'bake' looks for a target called 'main' automatically if
no parameters are provided on the command line. You can add a target called
'main' with a single dependency so 'bake' will do what you expect like this:
```
main=video.elf
```

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

## How to Debug
This is actually two different questions - (1) how to debug your recipe, and
(2) how to debug Bake itself. Fortunately, both of these things are just Bash
scripts, which means you can inspect the source code with any text editor if
it comes to that. But before jumping the gun, there are other things that you
can try.

### Print Out Your Variables
When you and bash disagree on what the contents of a variable are, it
invariably leads to a misunderstanding that manifests as a misbehaviour
(AKA bug). Make sure you and bash agree by using 'echo' to dump out the value
of the variable at various points in your script. Some common mistakes are
listed below.

1. Variable name masking (scope)
   Variables are generally global in bash. Even when you are inside a function
   you still have access to all of the global variables. If you want to reuse
   a variable name, you have to use the keyword 'local'.
2. Array variables
   When you reference an array variable, make sure you use the correct syntax.
   For example, if you have a list called 'c_src', referencing it as $c_src
   is legal, but will only reference the first item in the list. To get all of
   the items, you must write it as: ${c_src[@]}
   
### Use the -v option
If 'bake' is invoked with the -v flag, it will print out all of the decisions
it is taking so that you can trace through the execution.


[1]: bake#L24
