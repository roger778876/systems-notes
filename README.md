## 11/9 Time to make an executive decision.
**execlp & execvp**

**exec family - <unistd.h>**
- there are many C functions that can be used to run other programs from within
- exec replaces the current process with the new program; after exec, current program is gone

**execlp(NAME OF FILE>, <PROGRAM, ARGUMENTS, ...>, NULL)**
  - PROGRAM should == NAME OF FILE
  - ex: ```execlp("ls", "ls", "-a", NULL)```)
  - rest of current file will not run anymore
  - the "p" in execlp means PATH; will search for program in system's PATH; you can also put full file path in NAME OF FILE
  - new command will completely replace original program; keeps same PID
  
**execvp(NAME OF FILE>, <ARGUMENT ARRAY)**
  - first argument is still program file
  - argument array is an array of strings
  - last argument must be NULL
  - ex: ```char *args[5];
  
           args[0] = "ls";
           
           args[1] = "-a";
           
           args[2] = NULL;
           
           execvp(args[0], args)```
  

## 11/8 Sending mixed signals
Sometimes you need to send info to processes; we can use Signals

**Signals**
- limited way of sending info to a process
- sends an integer value to a process
- ex: ctrl-c to interrupt a program (SIGINT)
- ```man 3 signal``` to see list of signal numbers (1-31)
  - 30 & 31, (SIGUSR1 & SIGUSR2) have been put aside for user use

**kill**
- command line utility to send a signal to a process
- ```kill <PID>``` by default sends signal 15 (SIGTERM) to PID; terminates process PID
- can use different signal # to send different signal
  - ex: ```kill -<SIGNAL #> <PID>``` 

**killall**
- ```killall [-<SIGNAL #>] <PROCESS>```
- sends signal to all processes with PROCESS as the name

**Signal handling in C programs - <signal.h>**
- **kill**
  - ```kill(<PID>, <SIGNAL #>)```
    - returns 0 on success or -1 (errno) on failure
- reasons to catch/intercept a signal:
  - shut things down gracefully
  - not interrupt writing to a file
- **sighandler**
  - to intercept signals in C you must create a signal handling function
  - some signals can't be caught (ex: SIGKILL 9)
  - ```static void sighandler(int signo){...}```
    - must be static, void, and take single int parameter
    - static functions don't get put onto stack; separate area of memory
    - static functions can only be called from within the same file
      - therefore sighandler must be in same file as "main"
    - ```if (signo == <SIGNAL>) {...}``` inside sighandler
      - we need to attach <SIGNAL> to this function
        - put ```signal(<SIGNAL>, sighandler)``` in main
        - for every signal you want to catch, you need both if (...) and signal(...)
      - this will override the default signal action with {...}


## 11/6 Aim: Are your processes running? Then you should go out and catch them!
scanf is prone to user input errors, so we can use fgets

**fgets - <stdio.h>**
- reads in from a file stream and stores in a string
- ex: ```fgets(<DEST>, <BYTES>, <FILE POINTER>)```
  - reads at most BYTES - 1 characters from file stream FILEPOINTER and stores into DEST
- automatically puts in terminating NULL at end (hence why BYTES - 1 characters)
- stops at newline, end of file, or the byte limit, whichever comes first
  - similar to read line function
- if newline character fits into byte limit, it will keep the newline character as part of the string & appends NULL after
- ex: ```fgets(s, 256, stdin)```
  - reads a line of text from stdin/user input
- downside to fgets:
  - it reads in a string; so we can use sscanf
  
**FILE POINTER**
  - ```FILE * ``` type, more complex than a file descriptor
  - ```stdin``` is a FILE * variable
  - ```fopen(...)``` returns file pointer instead of descriptor
  
**Other**
- "blocking": when program is waiting for something to happen before proceeding (ex. stdin/user input)

**sscanf - <stdio.h>**
- scans a string and extracts values based on a format string
- ex: ```sscanf(<SOURCE STRING>, <FORMAT STRING>, <VAR1>, <VAR2>, ...)```
  - ex: ```sscanf(s, "%d %f", %i, %f);``` ; expects integer space floatingpoint
  
**Processes**
- every running program is a process
  - processes can create subprocesses, but they're the same as the original processes
- processor can handle 1 process per cycle (per core)
- "multitasking" seems to happen because the processor switches between all active processes so quickly

**pid**
- every process has unique identifier called pid
- pid 1 is the init process; stopping pid 1 will stop OS
- to see running processes, run ```ps``` in command line
  - by default, you'll only see processes that you are owner of and processes attached to terminal sessions (TTY #)
  - ```ps -a``` will see all processes attached to terminals
  - ```ps -aux``` will see all all processes
