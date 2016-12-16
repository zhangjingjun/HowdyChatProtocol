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


