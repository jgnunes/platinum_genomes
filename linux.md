# Table of contents  
1. [Basic concepts](#basic-concepts)  
    1.1 [The terminal](#the-terminal)  
    1.2 [The prompt](#the-prompt)  
    1.3 [Running commands](#running-commands)  
    1.4 [The shell](#the-shell)
2. [Navigating the system](#navigating-the-system)  
    2.1 [Where are we?](#where-are-we?)  
    2.2 [What's in our current directory?](#what's-in-our-current-directory?)  
    2.3 [Moving around](#moving-around)  
    2.4 [Shortcuts](#shortcuts)  
    2.5 [The tab completion trick](#the-tab-completion-trick)
3. [File manipulation](#file-manipulation)  
    3.1 [Creating a directory]("#creating-a-directory")  
    3.2 [Creating a file]("#creating-a-file")  
    3.3 [Copying a file]("#copying-a-file")  
    3.4 [Copying a directory]("#copying-a-directory")  
    3.5 [Moving files and directories]("#moving-files-and-directories")  
    3.6 [Renaming files and directories]("#renaming-files-and-directories")  
    3.7 [Removing a file]("#removing-a-file")  
    3.8 [Removing a directory]("#removing-a-directory")    
[Additional references](#additional-references)

# 1. Basic concepts <a name="basic-concepts"></a>  
## 1.1 The terminal <a name="the-terminal"></a> 
The terminal, or command line, is an interface that allows you to interact with the system through text. It allows the user to run functions or programs, open and browse through directories, see the processes that are currently running, among other things.  

## 1.2 The prompt <a name="the-prompt"></a> 
When you open the terminal, you will see something that looks like this (the exactly text may change depending on the system you are using, but they should all look similar):  

```console  
user@bash:~$ 
```

This is called a *prompt* and it means the terminal is ready for you to execute a command.  

## 1.3 Running commands <a name="running-commands"></a>  
When running a command, typically the command itself is the first thing you will type. After that you will type the arguments, which are the input information that will be fed into the command. Some commands also offer options to change its default behaviour. They are usually encoded by a dash (-) and placed before the arguments.  

Let's check a real example:  

```console
username@bash:~$ ls -l /home/username  
total 2  
drwxr-xr-x 18 username users 4096 Feb 17 09:12 Documents
drwxr-xr-x  2 username users 4096 May 05 17:25 public_html  
username@bash:~$
```  

The first line is the command execution. Here we are running the command **ls**, which lists all files and directories in the present working directory. This is probably one of the commands you will use the most, and we will discuss it in more detail later.  
Right after *ls* we see the option **-l**, which modifies the default behaviour of *ls* and forces it to print the output in long format, which displays more information about each object of the directory.  
The last element of the first line is the argument that is given to the command, which in this case is the path of the directory (**/home/user**) whose content we want to inspect.  

Lines 2-4 show the output of the command. Don't worry about understanding it now, since we will discuss the command *ls* in more detail later. Also notice that not every command returns an output. Some just do their job quietly and only display a message if an error has occurred.  

Finally, line 5 presents the user with the prompt again. That means the command execution is done and the terminal is ready to run another command.  

## 1.4 The shell  <a name="the-shell"></a> 
Whenever we run a command in the terminal, the command is processed by a program called *shell*, which will interpret the command and send back the output to the terminal. There are various shells available but the most common one is called *bash*, which stands for Bourne again shell. Bash is the most popular flavour of shell in Linux systems, and it also is the default shell on macOS. Over our course it is the shell we will be using.    

# 2. Navigating the system  <a name="navigating-the-system"></a> 
## 2.1 Where are we?  <a name="where-are-we?"></a> 
The command **pwd** stands for *Print Working Directory* and it will print the complete path to our current directory:  
```console  
username@bash:~$ pwd  
/home/username
```  

## 2.2 What's in our current directory?  <a name="what's-in-our-current-directory?"></a> 
The command **ls** will print all files and folders located in our current directory:  
```console  
username@bash:~$ ls  
Documents public_html
```  
If we want to print the content of a directory other than our current one, we need to give its path as an argument to ls:  
```  
username@bash:~$ ls /etc  
a2ps.cfg aliases alsa.d 
```  
If we want ls to print more information, we can use some of its options (remember, they will be preceded by a dash (-) symbol). For instance, option **-l** can print information on user permissions over that object, as well as its size and last modification date:  
```console  
username@bash:~$ ls -l /etc
-rwxr-xr-x  2 root root 1000000 Mar 23 13:34 a2ps.cfg
-rwxr-xr-x 18 root root 78 Feb 17 09:12 aliases
drwxr-xr-x  2 ryan users 4096 May 05 17:25 alsa.d
```  
Our last command has printed file sizes in bytes. For most cases we will want that in a human-readable format, what can be achieved by adding the **-h** option:  
```console  
username@bash:~$ ls -lh /etc
-rwxr-xr-x  2 root root 1M Mar 23 13:34 a2ps.cfg
-rwxr-xr-x 18 root root 78 Feb 17 09:12 aliases
drwxr-xr-x  2 ryan users 4,1K May 05 17:25 alsa.d
```  

If you want more information on the **ls** command (and any other command of the system), use the command **man**:  
```console  
username@bash:~$ man ls
```  
The command will print pretty useful information, such as how to execute the command, what are the options available and the arguments required (if any). 

## 2.3 Moving around  <a name="moving-around"></a> 
The command **cd** (which stands for *Change Directory*) allows us to move to another directory. Its syntax is straigh-forward since it only takes one argument, which is the target directory we want to move to:  
```console  
username@bash:~$ cd Documents  
username@bash:~$ pwd  
/home/username/Documents
``` 
In the last command we used **cd** to move to the *Documents* directory, which is a subdirectory of our previous directory (*/home/username*). In this case we have typed a *relative* path to inform we wanted to move to the target directory. Another possibility could have been to inform the *absolute* path to the directory, i.e, the complete path to it:  
```console  
username@bash:~$ cd /home/username/Documents
username@bash:~$ pwd  
/home/username/Documents
``` 

## 2.4 Shortcuts <a name="shortcuts"></a> 
If you want to go back to the previous (parent) directory, you don't need to specify its complete path. The shortcut **..** will do that for you:  
```console  
username@bash:~$ cd ..  
username@bash:~$ pwd  
/home/username
```  
You can even concatenate multiple **..**:  
```console    
username@bash:~$ cd ../..  
username@bash:~$ pwd  
/
```  
Now we are in the so called **root** directory. This is the top of the hierarchical structure of the system. All other directories are actually subdirectories of it.  

Another possibly useful shortcut is **~**, which represents the home directory of your user:  
```console  
username@bash:~$ cd ~  
username@bash:~$ pwd  
/home/username
```  

**Cool fact**: the shortcuts **..** and **~** also work with other commands besides **cd**:  
```console  
username@bash:~$ ls ~  
Documents public_html
```  

## 2.5 The tab completion trick <a name="the-tab-completion-trick"></a>  
Typing out long paths can be tedious and slow, without mentioning the chances of typing errors (typos). However, the command line have a powerfull mechanism to help us with that: it's called **tab completion**.  

The idea is that whenever you start typing a path, if you hit the Tab key on your keyboard the command line will invoke an autocompletion action. It nothing happens, it means there are several possibilities for autocompletion. In that case you should hit Tab again to show all the possibilites, continue typing and then hit Tab again to continue the autocompletion process. Try it yourself to seed the power of tab completion!

# 3. File manipulation <a name="file-manipulation"></a>  
## 3.1 Creating a directory <a name="creating-a-directory"></a>  
You can create a directory inside your current directory using the command **mkdir** (which stands for *Make Directory*):  
```console  
username@bash:~$ mkdir linux_tutorial 
username@bash:~$ ls  
Documents linux_tutorial public_html
```  
The execution above has created a directory inside our current working directory, which was */home/username*. However we can use **mkdir** to create a directory in any other location. Let's suppose we wanted to create a directory in the Documents directory instead:  
```console  
username@bash:~$ mkdir Documents/linux_tutorial_2 
username@bash:~$ ls  
Documents linux_tutorial public_html  
username@bash:~$ ls Documents  
linux_tutorial_2
```  
We can also create directories using absolute paths:  
```console  
username@bash:~$ mkdir /home/username/Documents/linux_tutorial_3 
username@bash:~$ ls Documents  
linux_tutorial_2 linux_tutorial_3
```  
## 3.2 Creating a file <a name="creating-a-file"></a> 
We can use the command **touch** to create an empty file:  
```console  
username@bash:~$ touch empty_file 
username@bash:~$ ls   
Documents empty_file linux_tutorial public_html
``` 
Do not worry: later on in this tutorial you will learn how to put data into the files you have created.  

## 3.3 Copying a file <a name="copying-a-file"></a>  
There are several reasons why you may want to copy a file, for instance when you want to create a backup copy before modifying the original copy. The command to create copies is **cp**. It takes two arguments: the first one is the file to be copied (i.e. the source) and the second is the destination. 

The destination is a path to either a file or a directory. If destination is a directory, then the **cp** command will create a copy of the source file in the destination directory and keep the original filename of the source file:  
```console  
username@bash:~$ cp empty_file linux_tutorial 
username@bash:~$ ls linux_tutorial  
empty_file
``` 
However, if the destination is a file, the **cp** command will create a copy of the source file with the new filename specified. This will be necessary when you want the copied file to have a different name:    
```console  
username@bash:~$ cp empty_file linux_tutorial/empty_file_backup 
username@bash:~$ ls linux_tutorial  
empty_file empty_file_backup
```  

## 3.4 Copying a directory <a name="copying-a-directory"></a>  
We will use the same **cp** command to copy an entire directory, however now we need to specify the *-r* option, which will indicate we want to copy the source directory and all of its files and subdirectories:  
```console  
username@bash:~$ cp -r linux_tutorial linux_tutorial_backup 
username@bash:~$ ls   
Documents empty_file linux_tutorial linux_tutorial_backup public_html  
username@bash:~$ ls linux_tutorial_backup  
empty_file empty_file_backup
```  

## 3.5 Moving files and directories <a name="moving-files-and-directories"></a>  
The command to move a file or directory from one directory to another is **mv** (which stands for *Move*).**mv** works in similar way as **cp**, where the first argument is the source and the second one is the destination. One advantage of **mv** over **cp** is that **mv** do not need the option *-r* to move directories, so the command is actually the same either you want to move a file or a directory:  
```console  
username@bash:~$ mv linux_tutorial/empty_file Documents
username@bash:~$ ls Documents  
empty_file linux_tutorial_2 linux_tutorial_3 
username@bash:~$ ls linux_tutorial  
empty_file_backup
```  
So in the command above we have moved the file *empty_file* from the *linux_tutorial* to the *Documents* directory.  

## 3.6 Renaming files and directories <a name="renaming-files-and-directories"></a>  
The command **mv** can also be used to rename a file or directory. This is accomplished when the source and destination directories are the same:  
```console  
username@bash:~$ ls linux_tutorial
empty_file_backup  
username@bash:~$ mv linux_tutorial/empty_file_backup linux_tutorial/empty_file_backup_renamed 
username@bash:~$ ls linux_tutorial  
empty_file_backup_renamed
```  
```console  
username@bash:~$ ls  
Documents empty_file linux_tutorial linux_tutorial_backup public_html
username@bash:~$ mv public_html public_html_renamed 
username@bash:~$ ls  
Documents empty_file linux_tutorial linux_tutorial_backup public_html_renamed
```  
In the first example, the file *empty_file_backup*, inside the directory *linux_tutorial*, is renamed to *empty_file_backup_renamed*.  
In the second example, the directory *public_html*, inside our current directory, is renamed to *public_html_renamed*.

## 3.7 Removing a file <a name="removing-a-file"></a>  
The command to remove a file is **rm**, which stands for *Remove*. The only argument it takes is the path to the file to be removed:  
```console  
username@bash:~$ ls linux_tutorial_backup  
empty_file empty_file_backup 
username@bash:~$ rm linux_tutorial/empty_file_backup  
username@bash:~$ ls linux_tutorial_backup  
empty_file
```  

## 3.8 Removing a directory <a name="removing-a-directory"></a> 
The command **rm** can also be used to remove entire directories, however just like for the command **cp**, we need to add the *-r* option:  
```console  
username@bash:~$ ls  
Documents empty_file linux_tutorial linux_tutorial_backup public_html
username@bash:~$ rm -r linux_tutorial_backup 
username@bash:~$ ls  
Documents empty_file linux_tutorial public_html_renamed
``` 

### Attention  
Since Linux command line does **not** have an **undo** option, be **careful** when performing **destructive** actions. 

# Additional references <a name="additional-references"></a>  
This tutorial was inspired by the great [Ryan Chadwick's webpage](https://ryanstutorials.net/linuxtutorial/) on Linux. If you want to deepen your understanding on Linux, I highly recommend you to browse through Ryan's tutorial, which includes exercises and some other interesting topics not coverede here due to time constraints.
