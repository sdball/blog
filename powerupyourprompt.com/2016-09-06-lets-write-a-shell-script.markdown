---
layout: single
title: "Let's write a shell script"
date: 2016-09-06T07:45:00-04:00
---

Now that you have a [$PATH](/blog/whats-a-path-anyway/) let's start using it!
Today we'll write a simple shell script in your personal `bin` directory.

Let's start by picking the language the shell script will be written in. You
can choose any scripting language that you have available on your machine. In
our script we'll tell the shell what command to execute to run the script.

## The shebang

The first line of a shell script is called the "shebang" or "hashbang". It
tells your shell that the way to run the script is to run that program with the
script file as an argument.

The shebang has a specific syntax: `#!/path/to/program/to/execute`

For example:

```shell
#!/bin/bash
```

Says to run this script as an argument to `/bin/bash`. If we had our script in
our home directory's bin folder then the `bash` command would be given the path
to that file as an argument.

```
/bin/bash /Users/sdball/bin/shell_script_file
```

This is a really awesome bit of tooling! It means we can write our "shell"
scripts in anything that we can call directly.

## Making a script executable

A key step when making your own shell script is that you need to allow your
shell to execute the script. You give specific files the permission to run as a
script with the special command `chmod`.

`chmod` can do a wide range of permissions modifications, but for our local
scripts we can keep it (relatively) simple. To make your scripts executable
run:

```shell
chmod +x script_filename
```

## Experiment!

Time to experiment! Let's make a file called `ls_script`.

```shell
#!/bin/ls

# this won't happen because ls doesn't know how to run commands
echo "hello"
```

This silly script will try to run with the `ls` command as its parser. If what
we've learned about the shebang is correct then what will _actually_ be run is
`/bin/ls /Users/sdball/bin/ls_script`. Since that file does really exist then
we should expect to see the `ls` command print out that file.

```shell
$ chmod +x ls_script
$ ls_script
/Users/sdball/bin/ls_script
```

It works! And interestingly we can see that the script actually runs slightly
differently depending on how we call it.

```shell
$ ls_script
/Users/sdball/bin/ls_script

$ cd $HOME/bin
$ ./ls_script
./ls_script
```

So we can give our script file to any command we can run. That means we can do
things like put `ruby` in the shebang and we'll end up having our shell
automatically run `ruby our_awesome_script`.

### Get weird!

How weird can we get? Let's change our `ls_script` file:

```
#!/bin/rm -f
```

```
$ ls_script
$ ls_script
zsh: command not found: ls_script
```

Well that worked. We just told our script to call `rm -f` with itself when it's
run. A neat vanishing act that can only happen once!

## Writing a real script

We've gotten enough of a grounding to kinda know what's going on in a shell
script. Let's write one!

But what should it do? How about we keep it simple and make a script called
`uppercase` that converts whatever we call it with into uppercase letters?

```shell
$ uppercase hello
HELLO

$ uppercase Hello, Shell
HELLO, SHELL

$ uppercase pi: 3.14159
PI: 3.14159
```

There is a unix command called `tr` that will be doing the bulk of the work for
us in this script. Our script will simply be wrapping a named command around
one use of it.

Like all the great unix tools, `tr` is designed to do one simple job. The
classic Unix idea being that you take a bunch of these tools that do one thing
and compose them into a data pipeline. It's a surprisingly effective concept,
but that's a topic for a future post.

The `tr` command "**tr**anslates characters". When given a set of characters
like "hello" it can be told to turn all the "l" characters into "1"s, or to
delete all the vowels, or shift each character up by one letter.

```shell
$ echo "hello" | tr l 1
he110

$ echo "hello" | tr -d aeiou
hll

$ echo "hello" | tr a-z b-za
ifmmp
```

The `tr` command typically takes two arguments: the characters to translate
(i.e. replace), followed by the character to translate into. Each character in
the first argument matches with its corresponding character in the second
argument.

```
# l translates into 1
$ echo "hello" | tr l 1
he110

# e translates into 3
# l translates into 1
# o translates into 0
$ echo "hello" | tr elo 310
h3110

# the range of a-y characters translates into b-z
# z translates into a
# (both of these commands are the same translation)
$ echo "hello abc xyz" | tr a-z b-za
ifmmp bcd yza

$ echo "hello abc xyz" | tr a-yz b-za
ifmmp bcd yza
```

In our script, we'll want `tr` to replace all lowercase letters with uppercase.

```
$ echo "hello" | tr a-z A-Z
HELLO
```

Let's write it! Open up your favorite text editor and create a new file called `uppercase`:

```shell
#!/bin/bash

echo $@ | tr a-z A-Z
```

That's it! Let's run it first and then we'll step through it.

```
$ chmod +x uppercase

$ ./uppercase hello 123
HELLO 123

$ ./uppercase Hello, Shell
HELLO, SHELL
```

It works! What does it do? Let's look at each part:

The shebang says to run this file with the `/bin/bash` command. That means
we're writing a bash script.

```shell
#!/bin/bash
```

`echo $@` says to echo out all of the the arguments that the command is given.
If we'd used, say, `echo $1` then only the first word would be processed.

```shell
echo $@
```

The `|` character says to "pipe" the data from the echo command into the
following command. At a low level: `STDOUT` from the first command is being
redirected into the `STDIN` of the second command.

```shell
|
```

Finally, our trusty `tr` command is being told to turn all the `a-z` characters
into their corresponding `A-Z` characters.

```shell
tr a-z A-Z
```

And we're done! You've written a bash shell script that actually does a thing!
It's a simple thing, sure, but it's better to have a bunch of scripts that do
one simple thing than to have one gigantic script that can do anything. After
all, if we had just one gigantic script then we might as well just use the `tr`
command directly.

Never underestimate the power of giving a name (e.g. `uppercase`) to a simple
piece of work.
