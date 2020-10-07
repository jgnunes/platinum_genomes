# The terminal  
The terminal, or command line, is an interface that allows you to interact with the system through text. It allows the user to run functions or programs, open and browse through directories, see the processes that are currently running, among other things.  

# The prompt 
When you open the terminal, you will see something that looks like this (the exactly text may change depending on the system you are using, but they should all look similar):  

```console  
user@bash:~$ 
```

This is called a *prompt* and it means the terminal is ready for you to execute a command.  

# Running commands  
When running a command, typically the command itself is the first thing you will type. After that you will type the arguments, which are the input information that will be fed into the command. Some commands also offer options to change its default behaviour. They are usually encoded by a dash (-) and placed before the arguments.  

Let's check a real example:  

```console
user@bash:~$ ls -l /home/user  
total 2  
drwxr-xr-x 18 ryan users 4096 Feb 17 09:12 Documents
drwxr-xr-x  2 ryan users 4096 May 05 17:25 public_html  
user@bash:~$
```  

The first line is the command execution. Here we are running the command **ls**, which lists all files and directories in the present working directory. This is probably one of the commands you will use the most, and we will discuss it in more detail later.  
Right after *ls* we see the option **-l**, which modifies the default behaviour of *ls* and forces it to print the output in long format, which displays more information about each object of the directory.  
The last element of the first line is the argument that is given to the command, which in this case is the path of the directory (**/home/user**) whose content we want to inspect.  

Lines 2-4 show the output of the command. Don't worry about understanding it now, since we will discuss the command *ls* in more detail later. Also notice that not every command returns an output. Some just do their job quietly and only display a message if an error has occurred.  

Finally, line 6 presents the user with the prompt again. That means the command execution is done and the terminal is ready to run another command.  

