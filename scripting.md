[Hangman]: http://www.cs.hmc.edu/courses/2014/fall/sysadmin/hangman_lab
[Dictionary]: http://www.cs.hmc.edu/courses/2014/fall/sysadmin/dict
[Permissions]: https:www.cs.hmc.edu/courses/2014/fall/sysadmin/linux_basics2.html

# Shell Scripting in Bash
*Originally created by Lisa Goeller, Joži McKiernan, and Paige Rinnert*

## Readings

Before beginning this lab, you'll probably want to do read up on shell
scripting. These readings come from the book *The Linux Command Line: A Complete
Introduction* by William Shotts.

**Strongly recommended**

  - Chapters 24-25 (p. 309--324) provide some background on setting and using 
    variables.
  - Chapters 26-29 (p. 325--362) discuss functions, if/else branching, keyboard 
    input, and while/until loops.

*Additional reading*

  - Chapter 31 (p. 375--380) talks about case branching, which will be useful 
    for the extra credit.

## Introduction

The goal of this lab is to write a program in bash to play Hangman! **Pair
programming is highly encouraged for this lab.** 

Bash is a scripting language. There are different kinds of shells (like sh, zsh,
and csh) but we will just be working in bash, which stands for **b**ourne
**a**gain **sh**ell (so named because it replaced sh, the original Bourne
shell).

To do this lab, you need to be familiar with constructing if/else statements and
while/until loops. You should also be fairly comfortable with regular
expressions. Don't be afraid to use Google to look up syntax that you are unsure
about! This [Bash cheat sheet](http://www.johnstowers.co.nz/blog/pages/bash-cheat-sheet.html) might also help.

## Materials

You'll need two files: [`hangman_lab`][Hangman] and [`dict`][Dictionary].  The
`hangman_lab` file contains the beginning of a script that implements hangman.
You'll modify this file as part of the lab. The `dict` file contains words that
the game can use; each line of the files contains a word.

There are two ways to get the files on your VM:

Download the files directly to your VM.
  : Log into your VM. In your home directory, use the `wget` command, along with
    the file URLs. Click on the links for the files above to visit the URLS, then
    copy the URL and run the command `wget <URL>`.

Download the files to your computer, then upload them to your VM.
  : Click the links for the files above, then save them to your computer. Use
    the `scp` command to upload them from your machine to your VM, like so: `scp
    <file> <username>@vm<#>.sys.cs.hmc.edu:~/`. This command will place the
    files in your home directory on your VM. Note: Your browser might try to add
    a .txt extension to the files. If so, you should remove the extension.

Notice that the `hangman_lab` file starts with a *"shebang"*:
```
#! /bin/bash
```
This line describes how to execute it: using the bash shell.

## Running a script

There are many ways to run a script or program on a *nix system. For this lab,
we'll run the hangman script as an executable file. To do so:

   1. Make sure the `hangman_lab` has executable permissions (see the [previous
   lab][Permissions] if you need a reminder for how to do this).
   1. Run the command `./hangman_lab`

In general, to run an executable file, you run the filename as if it were a
command. For example, `./hangman_lab` or `~/hangman_lab`. 

You can avoid typing the full path to the program, if the program's directory 
is in your path. The path is set by the `$PATH` variable in your environment. 
If the script is in a directory on your path, you will be able to run it by 
typing only the name of the script on the command line. For example, `ls`. 
To find out what directories are part of your path, type `echo $PATH` on
your command line. It will produce a colon-delimited list of directories
on your path. 

<!--
Throughout the process of writing the script, you can check periodically
that it is working by running it and seeing if the behavior is what you
expected. Be sure that any incomplete lines of code are commented out so
they don't return errors. Feel free to add print statements throughout
to help with debugging.
-->

## Defining Variables

We'll start implementing hangman by defining some variables. Look at the
function `define` in the `hangman_lab` file. This function defines some
variables for the program; you'll define some more. Here are the variables
already defined:

\/`word`
  : selected randomly from the provided dictionary using the command `shuf`,
    which shuffles the dictionary and returns the first line (a single word).

\/`length`
  : the length of a word

\/`wip`
  : stands for "word in progress". It is the game board. It begins as a sequence
    of underscores, each of which represents one letter of the word. The board
    gets filled in over the course of the game.

The variables you need to define are:

\/`lives`
  : the number of lives the player gets before they lose the game. In our
    version, we gave eight lives to the player. 

\/`gl`
  : stands for "guessed letters". It's an empty array that will
    eventually contain the letters that the player has guessed.

A few reminders about values and variables in bash:

  - An array, which is similar to a list, is represented by parentheses: `()`.
  - Whitespace is significant in bash. There shouldn't be any space between the 
    variable and its value, like this: `variable=value`.
  - To refer to refer to variables in bash, the syntax is `$variable`. If you 
    want to include a character directly after the variable (for
    example a period at the end of a sentence), include curly brackets
    around the variable: `${variable}`. 
  - To assign the results of a command to a variable (see, e.g., `word`, 
    `length`, and `wip`), the command needs to be enclosed in parentheses and 
    then preceded by a \$, like this: `variable=$(command)`.
  - To reference arrays, in addition to the \$ and curly brackets, you need to 
    tell bash to look at all elements of the array: `${array[@]}`. 

## Write `get_letter` to ask the player for a letter
_Note: Instead of writing the functions in the order they
will be called, which is the order they appear in the file, we encourage you to
write them in the order they appear in this lab description._

The `get_letter` function should ask the user to input a letter. There are three
parts to the function: first it prints a string to tell the user what to enter,
next it reads the user's input, and finally the function calls
`valid_letter` to determine if the input was valid.

Remember the print command in bash is `echo`. The command to ask for
user input is `read`, and the syntax is `read <variable>`. In this case,
we will call the variable `letter`. After adding code to a function,
it's okay to remove the `return` statements.

## Write `valid_letter` to validate the player's letter

Valid inputs consist of lowercase Roman alphabet characters only. If the
input is valid, the function should call the function `guessed_letter`.
(which we have provided because the syntax is kind of obscure.) If the input is 
invalid, the program should prompt the player for another letter, by calling 
`get_letter`. You might consider printing some additional helpful information 
for the player, such as a reminder of what letters have been guessed and what 
the word looks like now. 

<!--
## Write `guessed_letter` to check if the player's guess is new

We wrote this function, `guessed_letter`, for you because the syntax is
bizarre. The goal of this function is to check if the user has already
guessed the letter. If the user has guessed it, the function returns to
`get_letter` to ask for a different letter. If not, the function adds
the letter to the array `gl` which contains guessed letters, and it
calls `letter_in_word` to determine whether the letter is in the word.
-->

## Write `letter_in_word` to check the player's guess

The `letter_in_word` function checks whether the letter is in the secret word.
Use regular expressions to check whether the string `$word` contains the string
`$letter`. If it does, update the variable `$wip` to include the newly guessed
letter. To do this, you can use a slightly counterintuitive strategy: replace
all characters in `$word` NOT found in the guessed letters list with
underscores. The syntax of the find and replace command for strings in bash is
`${string/pattern/replacement}`. Use a regular expression to specify the 
pattern. When the player guesses a correct letter, you may also want to display
other information, e.g., the newly updated board and the letters that have been 
guessed.

If the player guesses a letter that's *not* in the word, the script should
decrement the number of lives.  You also need to call `check_lives` to determine
whether the user still has lives left or if the game is over. (You will write
this function later.)

## Write `game`, the main loop for the game

This function prompts the player for letters until the game is over. In order to
have an effective game, you first need to define your variables (funny, I think
you wrote a function for that already). You also need to let the player know how
long the word is by printing the board. Then, you need to keep asking for
letters until the word-in-progress is the same as the word itself. When the word
is guessed, the game should ask the user if they want to play again. 

## Write `check_lives` to check how many lives remain

If the player has no lives remaining, you should give them `$word` (just to be
nice) and the option to play again if they so wish. Otherwise, the player should
just keep playing. If the player decides to play again, both the word and the
variables need to be reset.

## Putting it all together

Now you're ready to run the script and test your game! You may have to think a
bit about how the functions tie together, but all the pieces are there!

## Extra Credit :)

If you still have some time left over, you can write a starting menu
that offers the option for the player to play in single-player mode or
against a friend. To write a menu, look at `case` branching (see Chapter
31 of the Linux book). It is within these branches that you should
define `$word`, whether it should be retrieved randomly from a
dictionary or whether it should be inputted into the game.

Also, if you are so inclined, you can try including ASCII art of a
hangman (or something less grotesque) that updates throughout the game.
Another idea is to create a cheating function that allows the player to
search through the dictionary with regex for possible matches to the
letters they've already guessed in the word. Play around, have fun, and
happy scripting!

