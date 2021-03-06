## Cisco in an hour IV: A new hope
**Transport Layer**
- computer to computer connection over a network
- unconcerned with the individual hops of layer 3 traffic
- uses network ports
- TCP and UDP are transport layer protocols (computer to computer)

**Application Layer**
- top most layer (everything else)

**Data Encapsulation**
- when data crosses from an upper layer to a lower layer, layer-specific metadata is added to help transmission
- Application > Transport; UDP/TCP headers are added, including network port info
- Transport > Network; data (including transport headers) is packaged into IP packets
- Network > Link; packets (including IP & transport headers) are packaged into Ethernet frames

**Data Decapsulation**
- when data crosses back up a layer, the packaging from the lower layer is removed


## 1/16 Cisco in an hour 3: In 3D
**Internet Layer**
Routing
- routers may break IPv4 packets into fragments
- When a router receives a packet, it has 2 options:
	1. Send that packet to the attached local network, OR
	2. Forward that packet to a different router
	
Differences between IPv4 & IPv6:
- address space (2^32 vs 2^128)
- packet format; IPv6 packet headers have less information, no checksum or fragment information
- MTU; IPv6 can allot for 2^32 (jumbograms)
- IPv6 puts more work onto Link Layer devices & individual hosts (computers)
	- doesn't fragment packets; relies on other devices to take advantage of jumbograms
	- has no checksum; assumes Link layer devices and hosts will check for data integrity if needed


## 1/12, 1/16 Cisco in an hour 2: Electric Boogaloo
**Internet Layer**
	- transmission of data between 2 separate networks
	- major features are addressing and routing
	- Routers are physical devices that connect different local networks
	- Internet layer traffic ignores the specifics of link layer traffic
	- IP Packets:
		- data sent over the internet layer is formatted into IP packets
		- IPv4 packet header:
			type,   size,   fragment info, ttl,   protocol, header checksum
			2bytes, 2bytes, 4bytes,        1byte, 1byte,    2bytes, 
			type: IPv4 or IPv6 & length of header
			size: total size of packet
			fragment info: full payloads can be broken into multiple fragments; each packet will count the # of fragments and its individual fragment number 
			ttl: time to live; max # of hops (routers) that a packet can make before reaching its destination
			protocol: TCP/UDP
			header checksum: only a checksum of the header, not the full packet
		- IPv4 Packet format:
			header , source, destination, data
			12bytes, 4bytes, 4bytes
			source & destination: IPv4 addresses
			data: MTU is 65,535 bytes


## 1/11 Cisco in an hour
- layer models of networking
	- network communications is often conceptualized into layers so people can work on specific components
	- the bottom layer is the most concrete, each layer above becomes more abstract (less physical, more code)
	- various competing models; ex: Open Systems Interconnections (OSI), TCP/IP
	
**TCP/IP Model layers**
1. Application Layer
2. Transport Layer
3. Internet
4. Link

**Link Layer**
	- point-to-point connection between devices on the same (local) network
	- combines physically connecting computers with basic addressing and transmission protocols
	- Physical connection:
		- how to transmit bits between 2 computers (ex: ethernet, radio frequencies, fiber optics)
		- Thicknet: a single coaxial cable runs along the network; vampire taps cut into cable & connect to computer
		- Thinnet: single coaxial cable;
		- problem is scalability
		- Ethernet: multiple computers connect to one hub/switch
			- Hub: broadcasts the data to all computers
			- Switch: sends data to a specific computer
	- in order for data to be sent between 2 computers:
		- each computer needs a unique MAC address
			- Media Access Control address
			- 6 byte hex address
			- MAC addresses only need to be unique on the same local network
		- data needs to be sent in a standardized format (Frames)
			- Ethernet Frames
			- each frame has format:
				prefix, dest,   source, type, data, checksum
				8bytes, 6bytes, 6bytes, 2bytes,   , 4 bytes
				- prefix: 10101010 x7 ... + 10101011
				- dest & source: MAC addresses
				- type: the type of frame
				- data: variable size depending on content; Max Transmission Unit (MTU) = 1500 bytes
				- checksum: ensures integrity of data


## 1/5, 1/8 Stop, Collaborate, and listen
- sockets are bidirectional

**To use a socket:**
1. Create the socket
2. Bind it to an address and port (server)
3. Listen and accept (server) or connect (client)
4. Send/Receive data

**socket <sys/socket.h>**
- creates a socket
- returns a socket descriptor (int that works like a file descriptor)
- ```socket(DOMAIN, TYPE, PROTOCOL)```
	- DOMAIN: type of address
		- AF_INET or AF_INET6 (ipv4 vs ipv6)
	- TYPE:
		- SOCK_STREAM or SOCK_DGRAM
	- PROTOCOL:
		- combination of domain and type settings
		- if set to 0, the operating system will set to correct protocol (TCP or UDP)
	- ex: ```int sd = socket(AF_INET, SOCK_STREAM, 0);```

**struct addrinfo**
- system library calls use ```struct addrinfo``` to represent network addresses
	- contain info like IP address, port, protocol, etc.
- ```c
		struct addrinfo
			.ai_family
				AF_INET: IPv4
				AF_INET6: IPv6
				AF_UNSPEC: IPv4 or IPv6
			.ai_socktype
				SOCK_STREAM
				SOCK_DGRAM
			.ai_flags
				AI_PASSIVE: automatically set to any incoming IP address
			.ai_addr
				pointer to a struct sockaddr containing the IP address
			.ai_addrlen
				size of the address in bytes
	```

**getaddrinfo (sys/types.h)(sys/socket.h)(netdb.h)**
- looks up info about the desired network address and get one or more matching struct addrinfo entries
- ```getaddrinfo(NODE, SERVICE, HINTS, RESULTS);```
	- NODE: 
		- string containing an IP address or hostname to look up
		- if NULL, will use the local machine's IP address
	- SERVICE:
		- string with a port number or service name (if the service is in /etc/services)
	- HINTS:
		- pointer to a struct addrinfo used to provide settings for the lookup (type of address, etc.)
	- RESULTS:
		- pointer to a struct addrinfo that will be a linked list containing entries for each matching address
- getaddrinfo will allocate memory for all these new structs created

**Using getaddrinfo**
```c
	struct addrinfo *hints, *results;
	hints = (struct addrinfo *) calloc(1, sizeof(struct addrinfo));
	hints->ai_family = AF_INET;
	hints->ai_socktype = SOCK_STREAM; // creates TCP socket
	hints->ai_flags = AI_PASSIVE; // only needed on server
	SERVER: getaddrinfo(NULL, "80", hints, &results); // server sets node to NULL
	CLIENT: getaddrinfo(" ip address ", " port number ", hints, &results);
	...
	free(hints);
	freeaddrinfo(results);
```

**bind (sys/socket.h)**
- for servers only
- returns 0 or -1 for error
- ```bind(SOCKET DESCRIPTOR, ADDRESS, ADDRESS LENGTH);```
	- SOCKET DESCRIPTOR: return value of socket
	- ADDRESS: pointer to a struct sockaddr representing the address
	- ADDRESS LENGTH: size of address in bytes
	- ADDRESS & ADDRESS LENGTH can be retrieved from getaddrinfo
	

**listen <sys/socket.h>**
- server only
- set a socket to passively await a connection
- needed for stream sockets
- does not block
- ```listen(SOCKET DESCRIPTOR, BACKLOG)```
	- SOCKET DESCRIPTOR: return value of socket
	- BACKLOG: 
		- number of connections that can be queued up
		- depending on the protocal, this may not do much

**accept <sys/socket.h>**
- server only
- accepts the next client in the queue of a socket in the listen state
- used for stream sockets
- performs the server side of the 3 way handshake
- cretes a new socket for communicating with the client, the listening socket is not modified
- returns a descriptor to the new socket
- blocks until a connection attempt is made
- ```accept(SOCKET DESCRIPTOR, ADDRESS, ADDRESS LENGTH)```
	- SOCKET DESCRIPTOR: descriptor for the listening socket
	- ADDRESS: 
		- pointer to a struct sockaddr_storage that will contain info about the new socket after accept succeeds
	- ADDRESS LENGTH:
		- pointer to a variable that will contain the size of the new socket address after accept succeeds
		
**Using listen and accept**
```c
	//create socket
	int sd;
	sd = socket(AF_INET, SOCK_STREAM, 0);
	
	//use getaddrinfo and bind
	
	int backlog = 10;
	listen(sd, backlog);
	
	int client_socket;
	socklen_t sock_size;
	struct sockaddr_storage client_address;
	
	client_socket = accept(sd, (struct sockaddr *)&client_address, &sock_size);
```

**connect <sys/socket.h> <sys/types.h>**
- client only
- connects to a stream currently in the listening state
- used for stream sockets
- performs the client side of the three way handshake
- binds the socket to an address and port
- blocks until a connection is made (or fails)
- ```connect(SOCKET DESCRIPTOR, ADDRESS, ADDRESS LENGTH)```
	- SOCKET DESCRIPTOR: descriptor for the socket
	- ADDRESS: pointer to a struct sockaddr representing the address
	- ADDRESS LENGTH: size of the address in ny bists
	- ADDRESS and ADDRESS LENGTH can be 
	- note that the arguments mirror those of bind()

```c
	//create socket
	int sd;
	sd = socket(AF_INET, SOCK_STREAM, O);
	
	struct addrinfo *hints, *results;
	//use getaddrinfo(...);
	
	connect(sd, results->ai_addr, results->ai_addrlen);
```
		
		
## 1/2, 1/3 Socket to me.

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
	- there are 2^128 IPv6 addresses
	
	
**Network Ports**
- allow a single computer to run multiple services
	- like a building with many apartments; building is IP, apartment is port
- a socket combines an IP address and port
- each computer has 2^16 (65,536) ports
- some ports are reserved for specific services
	- port 80: http
	- port 22: ssh
	- port 443: ssl
- you can select any port, as long as it won't conflict with a service running on the desired computer
	- ports < 1024 are reserved for system services and should generally not be used
	
**Network Connection Types**
- ***Stream Sockets***
	- reliable 2 way communication
	- must be connected on both ends
		- verifies with 3-way handshake
	- data is received in same order as it is sent (not as easy as it sounds)
	- most sockets use the Transmission Control Protocol (TCP)
- ***Datagram Sockets***
	- "Connectionless"; an established connection is not required
	- data sent may be received out of order (or not at all)
	- useful when speed is important; most media streaming services use datagram sockets; games 
	- uses the User Datagram Protocol (UDP)


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
