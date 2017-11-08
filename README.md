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
- every process has identifier called pid
- pid 1 is the init process; stopping pid 1 will stop OS
- to see running processes, run ```ps``` in command line
  - by default, you'll only see processes that you are owner of and processes attached to terminal sessions (TTY #)
  - ```ps -a``` will see all processes attached to terminals
  - ```ps -ax``` will see all all processes
