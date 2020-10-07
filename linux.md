# Table of contents  
1. [Basic concepts](#basic-concepts)  
    1.1 [The terminal](#the-terminal)  
    1.2 [The prompt](#the-prompt)
2. [Navigating the system](#navigating-the-system)


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

# Additional references <a name="additional-references"></a>  
This tutorial was inspired by the great Ryan Chadwick's webpage on [Linux](https://ryanstutorials.net/linuxtutorial/). If you want to deepen your understanding on Linux, I highly recommend you to browse through Ryan's tutorial, which includes exercises and some other interesting topics not coverede here for limitation of time.


