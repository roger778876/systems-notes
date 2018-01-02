## 2/2 Socket to me.

**Socket**
- a connection between 2 programs over a network
- a socket corresponds to an IP (internet protocol) address & port pair

- ***To use a socket***
	1. Create the socket
	2. Bind it to an IP address and port
	3. Listen/initiate a connection (ex: server/client)
	4. Send/receive data
	
**IP Addresses**
- all devices connected to the internet need to have an IP address
- IP addresses come in 2 types: IPv4 & IPv6
- addreses are allocated in blocks to make routing easier
- ***IPv4***: 4 byte addresses of the form:
	- [0-255].[0-255].[0-255].[0-255]
	- each group is called in octet
	- at most there are 2^32, ~4.3 billion IPv4 addresses
- ***IPv6***: 16 byte addresses of the form:
	- [0-ffff]:[0-ffff]:...:[0-ffff] (x8)
	- each group is known as a hextet
	- leading 0s are ignored
	- any number of consecutive all 0 hextets can be replaced with ::
		- ex: 0000:0000:0000:0000:004f:13c2:0009:a2d2
		- can be: :: 4f : 13c2 : 9 : a2d2
	- IPv4 addresses can be represented as 5 0-hextets, 1 ffff hextet, and the IPv4 address
		- 149.89.150.100 -> :: ffff: 139.89.150.100


## 12/18 Always tip your servers.
Use sighandlers to remove server's pipe after Ctrl-C.

How to handle multiple clients? The server can fork for every connection.

**Forking server/client design pattern**
- standard way of setting up server & multiple clients

- ***Setup***
- ***Handshake***
	- Client connects to server and sends the private FIFO name. Waits for a response.
	- Server receives client's message and forks off a ***subserver***
		- subserver will have access to private pipe & server pipe
	- Server closes and removes well-known pipe
	- Subserver connects to client FIFO & sends acknowledgement message
	- Client receives subserver message and removes its private pipe
- ***Operation***
	- Server recreates well-known pipe and waits for a new connection
	

## 12/15 Always tip your servers.
```c
	int server_handshake(...) {
		int from_client;
		char buffer[BUFFER SIZE];
		
		mkfifo("luigi", 0600);
		
		printf("server handshake: making wkp\n");
		from_client = open("luigi", 0_RDONLY, 0);
		read(from_client, buffer, sizeof(buffer));
		printf("server handshake: received [%s]\n", buffer);
		
		remove("luigi");
		
		*to_client = open(buffer, 0_WRONLY, 0);
		write(*to_client, buffer, sizeof(buffer));
		
		read(from_client, buffer, sizeof(buffer));
		printf("server handshake received);
		
		return from_client;
	}
	
	int client_handshake {
		int from_server;
		char buffer[BUFFER SIZE];
		
		*to_server = open("luigi", 0_WRONLY, 0);
		
		sprintf(buffer, "%d", getpid());
		mkfifo(buffer, 0600);
		
		write(*to_server, buffer, sizeof(buffer));
		
		from_server = open(buffer, 0_RDONLY, 0);
		read(from_server, buffer, sizeof(buffer));
		
		remove(buffer);
		
		write(*to_server, ACK,  sizeof(buffer));
		
		return 
	}
	
	client.c
	while (1) {
		fgets(buffer, sizeof(buffer), stdin);
		
		write(to_server, buffer, sizeof(buffer));
		read(from_server, buffer, sizeof(buffer));
		printf("received: [%s]\n", buffer);
	}
	
	server.c
	while (1) {
		from_client = server_handshake(&to_client);
		while (read(from_client, buffer, sizeof(buffer))) {
			PROCESSING DATA
			write(to_client, buffer, sizeof(buffer));
		}
	}
```

## 12/11 Creating a handshake agreement.
1. Client sends message to server (now the server knows it can receive data)
2. Server sends confirmation message (now the client knows it can send and receive data)
3. Client sends confirmation back (now the server knows it can send data)
aka a "3-way handshake" in program communications b/c it takes 3 steps

**Handshake**
  - a procedure to ensure that a connection has been established between 2 programs
  - both ends of the connection must verify that they can send and receive data to/from each other

**Basic server/client design pattern**
	- 2 named pipes between server & client
		- but this lacks security b/c anything could send data
	
- ***Setup***
	- Server creates a FIFO (Well known pipe) and waits for a connection
	- Client creates a "private" FIFO
- ***Handshake***
	- Client connects to server and sends the private FIFO name, then waits for response
	- Server receives the client's message and removes its well-known pipe
		- this will keep out other clients but maintains the connection
	- Server connects to client FIFO, sending an initial acknowledgement message
	- Client receives server's message and removes its private FIFO
	- Client sends response back to server
- ***Operation***
	- Server and client can now send information back and forth
- ***Reset***
	- Once the client terminates, server closes any connections to the client
	- Server recreates the well-known pipe and waits for a new client


## 12/7 What's a semaphore? To control resources!
```semctl()``` and Unions, ```semop()```

***```semctl()```***
- ***DATA***: variable for setting/storing semaphore metadata
- ```union semun```
  - you have to declare this union in your main c file on linux machines
  ```c
    union semun {
      int val; // for SETVAL
      struct semid_ds *buf; // for IPC_STAT and IPC_SET
      unsigned short *array; // for SETALL
      struct seminfo *__buf;
    };
  ```
    - you will only be using one of these values at a time
    - ***Unions***
      - a C structure designed to hold only one value at a time from a group of potential values
      - adding another value will remove the previously stored one
      - just large enough to hold the largest piece of data that it can potentially contain

- use ```ipcs -s``` in terminal to see current semaphores
- use ```ipcrm -s ID``` in terminal to remove semaphore ID

- ***```semop(DESCRIPTOR, OPERATION, AMOUNT)```***
  - perform atomic semaphore operations
  - you can Up/Down a semaphore by any integer value, not just 1
    - if you Down the semaphore below 0, it will block
  - ***DESCRIPTOR***: from semget()
  - ***OPERATION***: a pointer to ```struct sembuf```
    ```c
      struct sembuf {
        short sem_op;   // if Up, then any positive integer; if Down, then any negative integer;
                        // if 0, then block until the semaphore reaches 0
        short sem_num;  // the index of the semaphore that you want to perform on
        short sem_flag; // SEM_UNDO: allow the OS to undo the given operation; useful when a program exits before it could
                        // release a semaphore
                        // IPC_NOWAIT: instead of waiting for the semaphore to be available, return an err
      };
    ```
  - ***AMOUNT***: the amount of semaphores you want to operate on in the semaphore set
  

## 12/5 How do we flag down a resource?
How to control access to a shared resource (file, pipe, shared memory) such that no read/write conflicts can occur?
Use semaphores.

**Semaphores**
- created by Edsger Dijkstra
- IPC construct used to control access to a shared resource (file or shared memory)
- mostly commonly used as a counter representing how many processes can access a resource at a given time
  - ex: if a semaphore has value of 3, then it can have 3 active "users"
  - if a semaphore has a value of 0, then it is unavailable
- most semaphore operations are ***atomic***, aka they will not be split up into multiple processor instructions

**Semaphore Operations**
- Create a semaphore
- Set an initial value
- Remove a semaphore
- the above are not atomic
- Up(S) or V(S)
  - atomic
  - release the semaphore to signal you are done with its associated resource
  - Pseudocode: S++
- Down(S) or P(S)
  - atomic
  - attempt to take the semaphore
  - if semaphore value is 0, then waits for it to be available
  - Pseudocode: ```while (S == 0) { block } S--;```
  
- It is our job to attach semaphores to resources

**Semaphores in C**
- ```<sys/types.h>, <sys/ipc.h>, <sys/sem.h>```
- ***```semget(KEY, AMOUNT, FLAGS)```***
  - create/get access to a semaphore
  - not the same as Up(S) or Down(S); doesn't modify the semaphore
  - returns a semaphore int descriptor or -1 errno
  - ***KEY***: unique semaphore identifier
  - ***AMOUNT***: semaphores are stored as sets of one or more
    - specifies the # of semaphores to create/get in the set
  - ***FLAGS***: includes permissions for the semaphore; combine using bitwise or ( | )
    - IPC_CREAT: create the semaphore and set value to 0
    - IPC_EXCL: fail if the semaphore already exists and IPC_CREAT is on


## 12/1 Sharing is caring!, 12/4 Memes
Variables that are shared between parent and child

**Shared memory**
- <sys/shm.h>, <sys/ipc.h>, <sys/types.h>
- a segment of heap memory that can be accessed by multiple processes
  - you would have pointers in parent and child that refer to the shared memory
  - shared memory can be accessed from any process, not just parent/child
- shared memory is accessed via a ***key*** (integer) that is known by any process that needs to access it
  - similar purpose to a file name
  - to interact with the same shared memory, you just need the same key
- shared memory is ***not released*** when a program exits
  - only released when you say you don't want it anymore or reboot computer
  
**5 Shared memory operations**
- Create the segment (happens once in total)
  - you need to specify the size
  - ```shmget(KEY, SIZE, FLAGS)```
    - creates or accesses a shared memory segment; similar to opening a file (either creates or accesses)
    - returns a shared memory int descriptor (like a file descriptor) or -1 if errno
      - will be different from the key
    - KEY: unique integer identifier for the shared memory segment (like a file name)
      - a good place to put this is in header; ```#define KEY ___```; can be any int
    - SIZE: how much memory to request; in bytes
    - FLAGS: includes permissions for the segment, combined with bitwise or ( | )
      - ```IPC_CREAT```: create the segment; if segment is new, will set all values to 0
      - ```IPC_EXCL```: fail if the segment already exists and ```IPC_CREAT``` is on
    ```
      int sd;
      sd = shmget(KEY, sizeof(double), IPC_CREAT | 0600);
    ```
- Access the segment (happens once per process)
- Attach the segment to a variable (happens once per process)
  - then you'll get regular pointer access to the shared memory
  - ```shmat(DESCRIPTOR, ADDRESS, FLAGS)```
    - attach a shared memory segment to a variable
    - returns a pointer to the segment, or -1 if errno
    - DESCRIPTOR: the return value of shmget
    - ADDRESS: if 0, the OS will provide the appropriate address
      - just use 0
    - FLAGS: usually 0
      - ```SHM_RDONLY```: makes the memory read only (useful for processes with only read permissions)
    ```
      p = shmat(sd, 0, 0);
      *p = 222;
    ```
- Detach the segment from a variable (happens once per process)
  - ```c
    shmdt(POINTER)
    // returns 0 if successful, -1 if error
    ```
    - detach a variable from a shared memory segment
      - after detach, we will still have a number that points to a piece of memory we can't access
    - returns 0 if successful, or -1 if error
    - POINTER: address used to access the segment
- Remove the segment (happens once in total)
  - ```shmctl(DESCRIPTOR, COMMAND, BUFFER)```
    - perform operations on the shared memory segment
    - each shared memory segment has metadata that can be stored in a struct
      - ```struct shmid_ds```
      - ex: last access, size, pid of creator, pid of last modifier
    - DESCRIPTOR: return value of shmget
    - COMMAND: 
      - IPC_RMID: removes a shared memory segment
      - IPC_STAT: populate the BUFFER with segment metadata
      - IPC_SET: set some of the segment metadata from BUFFER
    - BUFFER: ```struct shmid_ds *```
- ```ipcs -m``` in terminal to list shared memory segments
- ```ipcrm -m SHMID``` to remove shared memory segment

## 11/28 C, the ultimate hipster, using # decades before it was cool
ex: ```#include <stdio.h>```

**#**
- used to provide preprocessor instructions
- aka: directives
- looked for & handled by gcc first; you can put it anywhere in the code, not just the top
- separate from C code

**#include**
- ```#include <LIBRARY>``` or ```#include "LIBRARY"```
- link libraries to your code
- means take all the contents of LIBRARY file and replace the #include line with it

**#define**
- ```#define <NAME> <VALUE>```
- like find & replace
- replaces all occurences of NAME with VALUE in the code
- don't include semicolons or equal signs!
- ```#define TRUE 1``` will replace every TRUE with 1
- similar to how O_RDONLY & O-WRONLY works
- ***macros***
  - like functions, but still just find & replace
  - ```#define SQUARE(x) x * x```
    - says anytime you see SQUARE and a value in parenthesis, replace it with value * value
    - ```int y = SQUARE(9);``` will become ```int y = 9 * 9;```
  - ```#define MIN(x, y) x < y ? x : y```
    - uses ternary operation to return minimum of x & y
- ***conditional statement***
  - useful for removing problem of double defining
  - use in header files
  - ```
       #ifndef <IDENTIFIER>
       ...CODE...
       #endif
    ``````
  - ifndef = if not defined
  - if the identifier has been defined, ignore all the code up to the endif statement
  - ex:
    ```
       #ifndef PARSE_H
        #define PARSE_H
        ...headers...
       #endif
       ```
    - this will stop from double defining things
    - this is how you should usually define things
    


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
