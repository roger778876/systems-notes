## 11/27 Redirection, how does it...SQUIRREL?

**File Redirection**
- changing the usual input/output behavior of a program

**Command line redirection**
- ```>```
  - redirects stdout to a file by overwriting
  - ex: ```ps > ps_file```; ```COMMAND > FILE```
  - ex: ```cat file1 > file2``` will copy contents of file1 to file2 (same as cp)
  - ex: ```cat ``` will just open stdin and print to stdin; typing something will just print it again
    - type end of file character to exit
    - ```cat > file``` will write input to file
- ```>>```
  - redirects stdout to a file by appending, not overwriting
- ```2>```
  - redirects stderr to a file by overwriting
  - ```2>>``` to append
  - useful for log files to keep track of errors; ex: for web servers or database servers
- ```&>```
  - redirects both stdout and stderr to a file by overwriting
  - ```&>>``` to append
  
- ```<```
  - redirects stdin from a file
  - useful for testing programs; if your program reads input, you can just make it read from a file
- ```|``` (pipe)
  - redirects stdout of one command into stdin of the next
  - ex: ```ls | wc``` will take output of ls and put into wc
  
**Redirection in code**
- ```dup``` - <unistd.h>
  - duplicates an existing entry in the file table
  - returns a new file descriptor for the duplicate entry
  - ```dup(FD)```
    - returns the new file descriptor
- ```dup2``` - <unistd.h>
  - ```dup2(FD1, FD2)```
  - redirects FD2 to FD1
  - duplicates the behavior for FD1 at FD2
    - replaces FD2 with FD1 in the file table
  - problem: you lose what FD2 was
    - fix by first dup(FD2), then dup2(); this will keep FD2 in reserve so you can undo it later
  

## 11/21 A pipe by any other name...
Named pipes aka FIFOs, first in first out (just like queues)

**Named pipes**
- same as unnamed pipes except they have a name that can be used to identify them via different programs
- unidirectional
- you just need to open a named pipe

**mkfifo PIPENAME**
- shell command to make a FIFO
- creates a pipe file; has the "p" permission
- ```cat mario``` & ```cat > mario```
- pipes only exist in memory; therefore, it has a size of 0 on the disk
  - still has entry in file table, just refers to place in memory
- you can write from more than one process; ex: use ```cat > mario``` twice
  - you can read from more than one process, but its random which one will read the pipe data
- as long as both ends of the pipe have connections, the pipe will stay open
- pipe will still work even if you delete the file
  - because the connection still exists in memory, not on the disk
  - basically turns into an unnamed pipe
  
**mkfifo - <sys/types.h> <sys/stat.h>**
- C function to create a FIFO
- returns 0 on success and -1 on error
- once created, the FIFO acts like a regular file
  - we can use open, read, write, close on it
- ```mkfifo(name, permissions)```
  - FIFOs will block on open until both ends of the pipe have a connection
    - must open on both ends of pipe


## 11/17 Ceci n'est pas une pipe
Pipes can let us pass information between processes

**Pipe**
- a conduit between 2 separate processes on the same computer
- have 2 ends: read and write
- unidirectional; a single pipe must be either read or write only in a process
- behave just like files
  - represented in file table
  - use file descriptors
- you can transfer any data you want through a pipe using read/write
- unnamed pipes have no external identifier
  - can't be identified outside of the program it was created in
  - like a secret connection; other programs can't identify it
  
**pipe - <unistd.h>**
- ```pipe(int DESCRIPTORS[2])```
  - creates unnamed pipe
  - returns 0 if pipe was created, -1 if error
  - opens both ends of the pipe as files for reading & writing
    - no particular read/write end when you create it; it's how you use it that specifies the end
  - DESCRIPTORS: array of 2 ints that will contain the file descriptors for both ends
- ```close(DESCRIPTORS[0 or 1])``` to close read or write function of end of pipe


## 11/15 Playing favorites
**endian-ness**
- byte order of values
- little endian vs. big endian
  - little endian: the smallest byte comes first
  - big endian: the largest byte comes first; ex: IP addresses
- ex: int x = 32;
  - little endian: 0 0 0 32
  - big endian: 32 0 0 0 
- if you write a data file on a big endian machine, then someone reads it on little endian machine, there can be problems
- endian-ness must be the same otherwise numbers will be rearranged

**wait & status**
- wait returns child PID
- first byte of status is 0 or signal value (when doing kill)
- second byte of status is return value (WEXITSTATUS(status))
- packing lots of info in a single integer (4 bytes)


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
  - argument array is an array of strings; each element is pointer to string literal, which is fine if you're not modifying them
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
