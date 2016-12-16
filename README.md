# HowdyChatProtocol

Howdy Chat Protocol is a simple application layer protocol running on TCP port number 20002, designed by Jingjun Zhang, to achieve basic information exchange between users who want to chat with each other online. Howdy Server and Howdy Client are developed on Python 3.5.2 platform as the implementation of Howdy Chat Protocol. The application is based on the following undelying technologies:

•	Socket programming to establish and control TCP connection as lower layer link between server and clients.

•	Multiple threads to achieve multiple client connections on server end and separation of sending and receiving on client end.

•	A simple application layer network protocol to divide phases of communication, negotiate username and distinguish, assemble and parse different type of messages. In this case, I designed a protocol named Howdy Chat Protocol.

•	A GUI development component to provide user interface for clients.

## Client-Server System

The system is composed of a server, multiple clients and HCP message sent and received by them. The server will be running one listening thread to accept new incoming connections and multiple handling threads to handle communications with each connection. While on client side, there will be two threads, one for sending messages, and the other for receiving message. Server’s socket will be set to allow reuse to accept multiple client. The detail of this system and application will be illustrated in the following chapters.

![System](/img/System.png)

## HCP Overview

HCP is running on TCP port number 20002 by default, the reliability of connection can be guaranteed by TCP. The communication of HCP is divided into two phases, Phase 1 for name negotiation and Phase 2 for message exchange. And in these two phases, there will be 5 different type of messages are exchanged between server and clients. Type 1 is ‘proposal’ message, sent from client to server with proposed username, to inform a possible username. Type 2 is ‘acknowledged’ message from server with proposed username, to inform it is accepted. Type 3 is ‘not acknowledged’ message from server to inform the proposed username is rejected. Type 4 is ‘update’ message from server to inform the newest user list on server, and it is triggered by event, like user logon and logoff. Type 5 is ‘message’ message, it could be send either from server to client or from client to server. Type 5 message carries text of conversation between users.

In phase 1, client will send Type 1 message to server. If the proposed username is accepted, server will send back a Type 2 message to confirm the username then update client with current user list, which means the end of Phase 1 and start of Phase 2, then online users can talk with each other. Otherwise, server will send a Type 3 message to reject the username explicitly, which means HCP stays in Phase 1.

In Phase 2, clients can send Type 5 message to server and server will forward Type 5 message to all other clients or one specific client according the receiver of message. Whenever there is an update of online user list, server will send Type 4 message to all online users to refresh user list on clients.

![HCP interaction](/img/HCP interaction.png)

Message type is represented with 1 byte length, and valid value will be 0x01 to 0x05. Length of username in HCP message is 64-bytes. For message Type 1 to 3, the format of message should be message type followed by proposed username. For message Type 4, the format message should be message type followed by a list of usernames, each of which will be 64-bytes long. For message Type 5, the format of message should be message type followed by username of sender, username of receiver and original message. ‘All’ and ‘Server’ is reserved as username for all the online user, so no one can choose them as usernames.

The following picture is the format of HCP messages:

![HCP packet format](/img/HCP packet format.png)

There is no consideration for disconnection since HCP can rely on TCP to handle it.

## HCP Server
The major ideas about design of Howdy server includes:
•	Keep server socket opening to listen new incoming client connections. Allow server socket to reuse IP address and port number of server.
•	For each new incoming client connection, create a new thread to receive HCP message from this client, handle or forward messages according type and receiver of HCP messages, and deal with username negotiation and change of connection status.
•	Maintain a dictionary for mapping between online username and client connection, and make it accessible for each thread so that the threads can modify the dictionary once there is any change on connection status.

The flowchart of original listening thread and threads created for new client connection is shown as below. When the server is started, there will be a command line prompt asking for IP address and port number to run the HCP service. The Howdy server will use them to start a new socket and listen for new clients. When a client connects server, a new thread will be created to handle this client connection, and the server socket will keep listening for next client. For the new thread of client connection, it will check the connectivity of client socket. If connectivity is lost, it will refresh user list, inform all remaining clients and close socket. Otherwise it will check message received, see if the message is type 1 or 5, BTW server will discard messages other than type 1 or 5 silently. For message type 1, server will fetch username and check its availability and inform users. For message type 5, server will forward the message to proper destination.

Socket Flowchart:

![serverflowchartsocket](/img/serverflowchartsocket.png)

Message Processing Flowchart:

![serverflowchartmessage](/img/serverflowchartmessage.png)

Server CLI:

![serverlogin](/img/serverlogin.png)

Server Log:

![serveroutput](/img/serveroutput.png)

## HCP Client
The major ideas about design of Howdy client includes:
•	GUI will be launched as soon as the Howdy Client starts, and future message sending will use the same thread with GUI. To send any message, user have to specify a destination from user list.
•	User should send name to server but cannot start a new thread for receiving message until server send HCP type 2 message.
•	The receiving thread will only deal with received data and decide how to present them to user.

The flowchart on Howdy Client application is shown as below. There are two flowcharts for sending and receiving thread. Packet sending share a thread with GUI, and it get message from GUI. First of all, it has to get IP address and port number from user input, and will not proceed until it connects the Howdy Server successfully. The next step is to provide a valid username to negotiate with server, and no message can be sent or received until the username is set. Message to be sent will also be checked and it will not be sent until it is valid. For receiving thread, it will check the connectivity of sock first, and will close the socket if it is not available anymore. Then it will check the message type, it can only take message type 2, 3, 4 and 5. Howdy Client will ignore message type 1 and discard unrecognized packets silently. If it received a message type 2, it will update its own username; if it receives a message type 3, it will inform user to pick a new name; if it receives a message type 4, it will update local user list and compare it with older one to find who went online or offline.

Socket Flowchart:

![clientflowchart](/img/clientflowchart.png)

Message Processing Flowchart:

![clientflowchartmessage](/img/clientflowchartmessage.png)

Client GUI:

![clientUI](/img/clientUI.png)

## Test


System test is running on a remote Server, the environment is shown as below.

![testenv](/img/testenv.png)

4 Windows 7 guest machines are created on ESXi bare-mental host, and each of them are allocated with 2 CPUs and 4G memory. The management of virtual machines is based on vSphere Client and Remote Desktops.
Win71 is running as server and the rest of them is running as clients.

Design of test cases are based on user requirement of this project and all the requirements are verified in these test case. The following two lists are key requirement and how test cases cover these key requirement.
1. Multiple client.
2. Clients must be able to “whisper” to each other.
3. Clients must be able to choose a nickname.
4. Server operations should be printed out by the server.
5. The server must handle connections/disconnections without disruption of other services.
6. Clients must have unique nicknames, duplicates must be resolved before allowing a client
to be connected.
7. All clients must be informed of changes in the list of connected users.
8. A list of online users must be displayed.
9. Connection/disconnection actions of users must be displayed on clients.
10. The messages you send must also be displayed.
11. Must still be able to receive messages/actions while typing a message.
12. Clients must be able to disconnect without disrupting the server.

The detailed test result is included in project documentation.
