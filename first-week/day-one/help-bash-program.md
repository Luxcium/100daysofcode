# How to add a Help facility to your Bash program

> From a 20 Dec 2019 article by [David Both](https://opensource.com/users/dboth)

![Image by: Opensource.com](https://opensource.com/sites/default/files/styles/image-full-size/public/lead-images/bash_command_line.png?itok=k4z94W2U)

**Image by:** Opensource.com

## Why Help?

> While creating your Help facility, you will also learn about using functions and how to handle command-line options such as **-h**.

Even fairly simple Bash programs should have some sort of Help facility, even if it is fairly rudimentary. Many of the Bash shell programs I write are used so infrequently that I forget the exact syntax of the command I need. Others are so complex that I need to review the options and arguments even when I use them frequently.
Having a built-in Help function allows you to view those things without having to inspect the code itself. A good and complete Help facility is also a part of program documentation.

## About functions

Shell functions are lists of Bash program statements that are stored in the shell's environment and can be executed, like any other command, by typing their name at the command line. Shell functions may also be known as procedures or subroutines, depending upon which other programming language you are using.

Functions are called in scripts or from the command-line interface (CLI) by using their names, just as you would for any other command. In a CLI program or a script, the commands in the function execute when they are called, then the program flow sequence returns to the calling entity, and the next series of program statements in that entity executes.

The syntax of a function is:

```bash
    FunctionName(){program statements}
```

Explore this by creating a simple function at the CLI. (The function is stored in the shell environment for the shell instance in which it is created.) You are going to create a function called **hw**, which stands for _"hello world."_ Enter the following code at the CLI and press **Enter**. Then enter **hw** as you would any other shell command:

```bash
    [student@testvm1 ~]$ hw(){ echo "Hi there kiddo"; }
    [student@testvm1 ~]$ hw
    Hi there kiddo
    [student@testvm1 ~]$
```

OK, so I am a little tired of the standard _"Hello world"_ starter. Now, list all of the currently defined functions. There are a lot of them, so I am showing just the new **hw** function. When it is called from the command line or within a program, a function performs its programmed task and then exits and returns control to the calling entity, the command line, or the next Bash program statement in a script after the calling statement:

```bash

    [student@testvm1 ~]$ declare -f | less
    <snip>
    hw ()
    {
        echo "Hi there kiddo"
    }
    <snip>
```

Remove that function because you do not need it anymore. You can do that with the `unset` command:

```bash
    [student@testvm1 ~]$ unset -f hw ; hw
    bash: hw: command not found
    [student@testvm1 ~]$
```

## Creating the Help function

> - In the [first article](https://opensource.com/article/19/12/introduction-automation-bash-scripts) in this series, you created a very small, one-line Bash script and explored the reasons for creating shell scripts and why they are the most efficient option for the system administrator, rather than compiled programs.
> - In the [second article](https://opensource.com/article/19/12/creating-bash-script-template), you began the task of creating a fairly simple template that you can use as a starting point for other Bash programs, then explored ways to test it.

Open the **hello** program in an editor and add the Help function below to the **hello** program code after the copyright statement but before the **echo _"Hello world!"_** statement. This Help function will display a short description of the program, a syntax diagram, and short descriptions of the available options. Add a call to the Help function to test it and some comment lines that provide a visual demarcation between the functions and the main portion of the program:

```bash
    ################################################################################
    # Help                                                                         #
    ################################################################################
    Help()
    {
      # Display Help
      echo "Add description of the script functions here."
      echo
      echo "Syntax: scriptTemplate [-g|h|v|V]"
      echo "options:"
      echo "g     Print the GPL license notification."
      echo "h     Print this Help."
      echo "v     Verbose mode."
      echo "V     Print software version and exit."
      echo
    }

    ################################################################################
    ################################################################################
    # Main program                                                                 #
    ################################################################################
    ################################################################################

    Help
    echo "Hello world!"
```

The options described in this Help function are typical for the programs I write, although none are in the code yet. Run the program to test it:

```bash
[student@testvm1 ~]$ ./hello
Add description of the script functions here.

Syntax: scriptTemplate [-g|h|v|V]
options:
g     Print the GPL license notification.
h     Print this Help.
v     Verbose mode.
V     Print software version and exit.

Hello world!
[student@testvm1 ~]$
```

Because you have not added any logic to display Help only when you need it, the program will always display the Help. Since the function is working correctly, read on to add some logic to display the Help only when the **-h** option is used when you invoke the program at the command line.

## Handling options

A Bash script's ability to handle command-line options such as **-h** gives some powerful capabilities to direct the program and modify what it does. In the case of the **-h** option, you want the program to print the Help text to the terminal session and then quit without running the rest of the program. The ability to process options entered at the command line can be added to the Bash script using the `while` command (see [How to program with Bash: Loops](https://opensource.com/article/19/10/programming-bash-loops) to learn more about `while`) in conjunction with the `getops` and `case` commands.

The `getops` command reads any and all options specified at the command line and creates a list of those options. In the code below, the `while` command loops through the list of options by setting the variable `$options` for each. The `case` statement is used to evaluate each option in turn and execute the statements in the corresponding stanza. The `while` statement will continue to evaluate the list of options until they have all been processed or it encounters an exit statement, which terminates the program.

Be sure to delete the Help function call just before the **echo _"Hello world!"_** statement so that the main body of the program now looks like this:

```bash
################################################################################
################################################################################
# Main program                                                                 #
################################################################################
################################################################################
################################################################################
# Process the input options. Add options as needed.                            #
################################################################################
# Get the options
while getopts ":h" option; do
   case $option in
      h) # display Help
         Help
         exit;;
   esac
done

echo "Hello world!"
```

Notice the double semicolon at the end of the exit statement in the case option for **-h**. This is required for each option added to this case statement to delineate the end of each option.

## Testing

Testing is now a little more complex. You need to test your program with a number of different options —and no options— to see how it responds. First, test with no options to ensure that it prints **_"Hello world!"_** as it should:

```bash
[student@testvm1 ~]$ ./hello
Hello world!
```

That works, so now test the logic that displays the Help text:

```bash
[student@testvm1 ~]$ ./hello -h
Add description of the script functions here.

Syntax: scriptTemplate [-g|h|t|v|V]
options:
g     Print the GPL license notification.
h     Print this Help.
v     Verbose mode.
V     Print software version and exit.
```

That works as expected, so try some testing to see what happens when you enter some unexpected options:

```bash
[student@testvm1 ~]$ ./hello -x
Hello world!
[student@testvm1 ~]$ ./hello -q
Hello world!
[student@testvm1 ~]$ ./hello -lkjsahdf
Add description of the script functions here.

Syntax: scriptTemplate [-g|h|t|v|V]
options:
g     Print the GPL license notification.
h     Print this Help.
v     Verbose mode.
V     Print software version and exit.

[student@testvm1 ~]$
```

The program just ignores any options without specific responses without generating any errors. But notice the last entry (with **-lkjsahdf** for options): because there is an **h** in the list of options, the program recognizes it and prints the Help text. This testing has shown that the program doesn't have the ability to handle incorrect input and terminate the program if any is detected.

You can add another case stanza to the case statement to match any option that doesn't have an explicit match. This general case will match anything you have not provided a specific match for. The case statement now looks like this, with the catch-all match of **\?** as the last case. Any additional specific cases must precede this final one:

```bash
while getopts ":h" option; do
   case $option in
      h) # display Help
         Help
         exit;;
     \?) # incorrect option
         echo "Error: Invalid option"
         exit;;
   esac
done
```

Test the program again using the same options as before and see how it works now.

## Where you are

You have accomplished a good amount in this article by adding the capability to process command-line options and a Help procedure. Your Bash script now looks like this:

```bash
#!/usr/bin/bash
################################################################################
#                              scriptTemplate                                  #
#                                                                              #
# Use this template as the beginning of a new program. Place a short           #
# description of the script here.                                              #
#                                                                              #
# Change History                                                               #
# 11/11/2019  David Both    Original code. This is a template for creating     #
#                           new Bash shell scripts.                            #
#                           Add new history entries as needed.                 #
#                                                                              #
#                                                                              #
################################################################################
################################################################################
################################################################################
#                                                                              #
#  Copyright (C) 2007, 2019 David Both                                         #
#  LinuxGeek46@both.org                                                        #
#                                                                              #
#  This program is free software; you can redistribute it and/or modify        #
#  it under the terms of the GNU General Public License as published by        #
#  the Free Software Foundation; either version 2 of the License, or           #
#  (at your option) any later version.                                         #
#                                                                              #
#  This program is distributed in the hope that it will be useful,             #
#  but WITHOUT ANY WARRANTY; without even the implied warranty of              #
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the               #
#  GNU General Public License for more details.                                #
#                                                                              #
#  You should have received a copy of the GNU General Public License           #
#  along with this program; if not, write to the Free Software                 #
#  Foundation, Inc., 59 Temple Place, Suite 330, Boston, MA  02111-1307  USA   #
#                                                                              #
################################################################################
################################################################################
################################################################################

################################################################################
# Help                                                                         #
################################################################################
Help()
{
   # Display Help
   echo "Add description of the script functions here."
   echo
   echo "Syntax: scriptTemplate [-g|h|t|v|V]"
   echo "options:"
   echo "g     Print the GPL license notification."
   echo "h     Print this Help."
   echo "v     Verbose mode."
   echo "V     Print software version and exit."
   echo
}

################################################################################
################################################################################
# Main program                                                                 #
################################################################################
################################################################################
################################################################################
# Process the input options. Add options as needed.                            #
################################################################################
# Get the options
while getopts ":h" option; do
   case $option in
      h) # display Help
         Help
         exit;;
     \?) # incorrect option
         echo "Error: Invalid option"
         exit;;
   esac
done

echo "Hello world!"
```

Be sure to test this version of the program very thoroughly. Use random inputs and see what happens. You should also try testing valid and invalid options without using the dash (-) in front.

## Next time

In this article, you added a Help function as well as the ability to process command-line options to display it selectively. The program is getting a little more complex, so testing is becoming more important and requires more test paths in order to be complete.

The next article will look at initializing variables and doing a bit of sanity checking to ensure that the program will run under the correct set of conditions.

---

## Resources

> (From opensource.com)

- [How to program with Bash: Syntax and tools](https://opensource.com/article/19/10/programming-bash-syntax-tools)

- [How to program with Bash: Logical operators and shell expansions](https://opensource.com/article/19/10/programming-bash-logical-operators-shell-expansions)
- [How to program with Bash: Loops](https://opensource.com/article/19/10/programming-bash-loops)

### More on Bash

- [Bash cheat sheet](https://opensource.com/downloads/bash-cheat-sheet?intcmp=7013a000002CxqaAAC)
- [An introduction to programming with Bash](https://opensource.com/downloads/bash-programming-guide?intcmp=7013a000002CxqaAAC)
- [A sysadmin's guide to Bash scripting](https://opensource.com/downloads/bash-scripting-ebook?intcmp=7013a000002CxqaAAC)

- [Latest Bash articles](https://opensource.com/tags/bash?intcmp=7013a000002CxqaAAC)

> ## About the author
>
> David Both is an Open Source Software and GNU/Linux advocate, trainer, writer, and speaker who lives in Raleigh North Carolina. He is a strong proponent of and evangelist for the "Linux Philosophy." David has been in the IT industry for nearly 50 years. He has taught RHCE classes for Red Hat and has worked at MCI Worldcom, Cisco, and the State of North Carolina. He has been working with Linux and Open Source Software for over 20 years. David prefers to purchase the components and build his...

[Source: **`Ctrl`**+**`Insert`**`/`**`Shift`**+**`Insert`**](https://opensource.com/article/19/12/help-bash-program)

[Author: David Both ](https://opensource.com/users/dboth)

[![Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)](https://opensource.com/sites/all/themes/opensource/images/cc-by-sa-4.png)](https://creativecommons.org/licenses/by-sa/4.0/)

[Copyright David Both, Opensource.com, Red Hat, Inc. or others ©2019-2020](https://opensource.com/article/19/12/help-bash-program#block-os-common-components-footer-call-to-action)

---

## \#100DaysOfCode Chalenge day 1/100

> ### Edited by Luxcium © 2020 Benjamin Vincent Kasapoglu
>
> see above for the original autor and copyright infos of the source.
> I aspire to publish all my content under MIT license **_but may not be able to do so in all cases._** You are responsible for ensuring that you have the necessary permission to reuse any work on this page.
