# **LAB5_MULTITHREADING_23-24_TAKE-HOME**
## Important
- If you are facing issues such as inconsistent evaluation, and are using WSL, please try using a VM, native Linux or the CC lab machines.
## **Instructions**

- This lab will be evaluated automatically.

- Any strings which are required to be sent by the client or server must
  be exactly as specified. (This includes the number of bytes that are
  being sent)

- You'll receive a binary containing sample test cases, and your
  submission will be evaluated against hidden test cases (not the ones
  you will use for testing) to determine the final score. So refrain
  from hard coding solutions.

- Submit a zip file with the name `<id_no>_lab5.zip` containing your
  implementation of client and server. It should be a zip of this repo.
  Ensure the impl folder is present and contains server.c.

Example Submission of necessary files, (you may have extra folders/files, it wont change anything):
```
<id_no>_lab5.zip
└── impl
    └── server.c
```


<br>

- **To evaluate:** run `./server_final` and ensure you are in the `./eval` directory.
- You may also run `./server_final <wait_time>` in order to increase the sleep time after connections are initiated. (default of 3)

Please note the binary is meant to verify or evaluate your code, **not
for debugging it**. If you wish to use it to debug, do so at your own
risk.

The purpose of this lab is to implement a client and server program that
uses multi-threading to communicate. Which can eliminate the drawbacks
of blocking calls such as recv() and allows communication to occur in
parallel.

In the last lab, you would have observed that the server can only listen
to one client at a time. The server thus sends the POLL command to get
only one client’s data. Similarly, on the client end, it waits for the
POLL message and when it gets the POLL message, it gets user input. To
prevent such constraints, in this lab, you will implement a client that
can receive commands from server and user at any time, where one
listening does not block the other. Similarly, the server can listen to
and send messages to any client.

## **Take Home**

For this lab, implement both client and server code, they will both be
utilizing **multithreading** to send and receive messages in parallel.

### **client.c**

- The client will take in IP address, port through CLI
  arguments in the format: `./a.out <ip_address> <port>`
  and connect to the server.

- A `<username>` will be sent to the server as the first message after
  the server accepts the connection (Details in the example at the bottom). Note carefully that the client doesn't include `\n` along with the `<username>` sent!
  
- The client will be referred to by this `<username>`.

- The client will take input through stdin and send it to the server.

- The client will receive data from the server and print it to stdout.

- **Note:** that taking input from stdin and receiving data are both
  blocking calls. Ensure these 2 can run in parallel using
  multithreading.

- **Note:** The client will not be evaluated, however you need to
  implement it to test your server.

### **server.c**

- The server will take in IP address and port through CLI arguments in
  the format: `./a.out <ip_address> <port>` and run on that ip and
  port.

- The server will listen to a max of 1024 connections, although it can accept any amount of
  client connections. (Requirement for server evaluation)

- The server must be able to receive data from all clients in parallel
  using multithreading. **Prerequisite for further evaluation**

- The client will send messages that start with a 4 letter command. The server's behaviour will depend on this command.

- Based on the data received from the client, the server will perform
  different functions:

### **Commands available to client**

#### **`LIST\n` (0.5 marks)**

- When the server receives this command it sends a string containing the
  names of all the clients that are currently connected to it in the
  format `<name1>:<name2>:<name3>\n`

#### **`MSGC:<receiver_username>:<message>\n` (1 mark)**

- The server will read `<receiver_username>` and send `<message>\n` to
  that user in the format `<sender_username>:<message>\n`.

- If the user does not exist, the server will send `USER NOT FOUND\n` back
  to the sender.

#### **`GRPS:<user1>,<user2>,...,<usern>:<groupname>\n` (0.5 + 0.5 marks)**
- This command indicates that a client want to create a user-group with name `<groupname>`. This group will have the indicated users
   
- The server will create a group consisting of `<user1>,
  <user2>,...,<usern>` and it will be identified by `<groupname>`. **(0.5
  marks)**

- `<groupname>` will be a single word without spaces or special characters.

- On Successful group creation, the server sends back `GROUP <groupname> CREATED\N`

- If any of the listed users not available, then the server will return
  `INVALID USERS LIST\n` and would not create the group at all. **(0.5
  marks)**

#### **`MCST:<groupname>:<message>\n` (0.5 + 0.5 marks)**

- The server will send the `<sender_username>:<message>\n` to all users in the mentioned
  group. **(0.5 marks)**

- If the `<groupname>` does not exist, the server will send `GROUP <groupname> NOT FOUND\n` back to the sender. **(0.5 marks)**

- If any of the group members has already exited, just send the message
  to the existing members.

#### **`BCST:<message>\n` (0.5 marks)**

- The server will send the `<sender_username>:<message>\n` to all the connected clients
  excluding the client who sent the message.

#### **`HIST\n` (1 mark)**

- The server will return the entire chat history in the format:

- `<sender_username>-<message>` this message will be the exact data
  sent by the user. \n will be used as the delimiter between each
  sender's messages.

- E.g. if user bob sends the message `MSGC:alice:banana\n`, the history will
  show: `bob-MSGC:alice:banana`. With `\n` as the delimiter.
  
- Note: The `<username>` sent by a newly connected client will not be logged for history.

#### **`EXIT\n` (1 marks)**

- The server will stop listening for that client.

- **Note:** Ensure if a client sends `EXIT\n` it will no longer show in the
  `LIST\n` of clients.

If the server receives a message which is not a part of any of these
commands, send `INVALID COMMAND\n` back to the sender.

## Example
An example communication as observed in the client side (alice):

The comminication is from alice's POV, and will show messages from different users.
```
[cli] ./a.out 127.0.0.1 4444

[cli] alice\n

[cli] LIST\n

[data from server] alice:bob:eve\n

[data from server] eve:online?\n

[cli] MSGC:bob:hello\n

[cli] MSGC:eve:yes\n

[cli] MSGC:xxx:hello\n

[data from server] USER NOT FOUND\n

[cli] GRPS:alice,bob:goodOnes\n

[data from server] GROUP goodOnes CREATED\n

[cli] GRPS:alice,xxx:goodOnes\n

[data from server] INVALID USERS LIST\n

[cli] MCST:goodOnes:eveIsEvil\n

[data from server] eve:tryingToHack\n

[cli] MCST:goodS:evil\n

[data from server] GROUP goodS NOT FOUND\n

[cli] BCST:helloAll\n

[cli] HIST\n

[data from server]
alive-LIST\neve-MSGC:alice:online?\nalice-MSGC:bob:hello\nalice-MSGC:eve:yes\nalice-MSGC:xxx:hello\nalice-GRPS:alice,bob:goodOnes\nalice-GRPS:alice,xxx:goodOnes\nalice-MCST:goodOnes:eveIsEvil\neve-BCST:tryingToHack\nalice-MCST:goodS:evil\nalice-BCST:helloAll\nalice-HIST\n

[cli] EXT\n

[data from server] INVALID COMMAND

[cli] EXIT\n
```
