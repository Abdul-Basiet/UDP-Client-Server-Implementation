# UDP-Client-Server-Implementation

This project seeks to implement a simple UDP/TCP communication between Unity and IoT devices in a Mixed Reality set up. The main goal is to be able to control IoT devices from Unity so there would be series of implementations and updates to that effect. Each stage would be documented in details.
[Unity](https://en.wikipedia.org/wiki/Unity_(game_engine)) is a cross-platform engine for developing games and applications on different platforms including mobile,consoles,desktop and the 3 realities now - Virtual Reality(VR), Mixed Reality(MR) and Augment Reality(AR).

User Datagram Protocol(UDP)/Transmission Control Protocol(TCP) is a simple protocol at the transport layer of the Internet Protocol stack that makes the best effort to deliver datagrams to a remote host. It is a connectionless protocol, that is to say, it does not guarantee establishment of connection prior to communicating with the remote host. [RFC 768](https://tools.ietf.org/html/rfc768). However, TCP guarantees delivery of packet, ensures flow control and relaible as compared to UDP.[RFC 793](https://tools.ietf.org/html/rfc793#section-2.1)

The initial scope of this project includes the setting up of UDP or TCP server on a remote host and a UDP client in unity, send a simple "Hello World!!!" from the latter to the former to test UDP connection. The main scripting language for the client side would be in C# since that is the main language in Unity now. Server side is scripted in Python in the mean time. Subsequent implementation might change in terms of the Server-side scripting.
 
# Tools
* Unity v2018.4.9
    - Scripting Language: C#
* Pycharm 
    - Scripting Language: Python
* Client PC
* Server PC

# Setting up the Server (Local Host)
Server is expected to receive some packets from the Unity client, process them and sends a reply back to the Client. The server in this case was hosted locally on port 12000. 
Below is a simple python script used to implement a UDP server on my local machine.

```python
from socket import *
import time
serverPort = 12000  #server port
serverSocket = socket(AF_INET, SOCK_DGRAM) #UDP connection function
serverSocket.bind(('', serverPort)) #binds the serverIP to the port
print('The server is ready to receive')
while True:
    responseMsg_1="Hey there, I got this!!!"
    message, clientAddress = serverSocket.recvfrom(2048)
    print('Message received from {}:'.format(clientAddress),message.decode())
    if message.decode()=="Hello World!!!":
        serverSocket.sendto(responseMsg_1.encode(), clientAddress)
    else:
        serverSocket.sendto("You killed me!!!".encode(), clientAddress)
        time.sleep(2)
        exit()
```
# Setting up the UDP Client in Unity.
UDP Client is scripted in Unity with C# language with the aim of sending a simple "Hello World!!!" to the remote server. The UDP Server upon receiving the message will reply with "Hey there, I got this!!!". This is to confirm delivery packets to the server. For clarity, I employed some of the properties of Unity's UI engine to create and Text field to hold the message and SendUDP Button.
Reply messages from the server will be logged on the console as a debug log.

```C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using System;
using System.Net;
using System.Net.Sockets;
using System.Text;
using System.Threading;



public class SendUDP : MonoBehaviour
{
    Thread receiveThread;
    public InputField inputUdp;

    //Method to Initialise a thread for new connection
    private void init()
    {

        receiveThread = new Thread(ConGet)
        {
            IsBackground = true
        };
        receiveThread.Start();
    }

    //Send Datagram method
    private void ConGet()
    {

        try
        {
            UdpClient myClient = new UdpClient(1344); //an instance of the UdpClient Class myClient with an assigned port

           

                myClient.Connect("localhost", 12000);

                // Sends a message to the host to which you have connected.
                Byte[] sendBytes = Encoding.ASCII.GetBytes(Convert.ToString(inputUdp.text));

                myClient.Send(sendBytes, sendBytes.Length);



                //IPEndPoint object will allow us to read datagrams sent from any source.
                IPEndPoint remoteSvr = new IPEndPoint(IPAddress.Any, 0);

                // Blocks until a message returns on this socket from a remote server.
                Byte[] recvPackets = myClient.Receive(ref remoteSvr);
                string returnData = Encoding.ASCII.GetString(recvPackets);

                // Print the response received from remote server
                Debug.Log("Server Response: " + returnData.ToString());
                Debug.Log("From IP:" + remoteSvr.Address.ToString() +
                                            "\n\tPort Number: " + remoteSvr.Port.ToString());

            myClient.Close();
            

        }
        catch (Exception e)
        {
            print(e.ToString());
          
        }
            
    }

    ////End Send Datagram method

    // Stop sending method
    public void OnApplicationQuit()
    {  
        if (receiveThread!=null)
        {
            receiveThread.Abort();
            Thread.Sleep(0); //Thread dies out when no response was received after sent packet
        }

        print("Stop");
    }
    // Stop sending 


    public void Button_Click() //Button Click to Send Packets
    {

        init();     //Call Send Method
       

    }
}

```
The GUI below has two components as mentioned earlier. The Input field - takes message and the Button labelled as **SendUDP** sends the message to the client upon a click.
![GUI](https://github.com/Abdul-Basiet/UDP-Client-Server-Implementation/blob/master/GUI_UDPClient.JPG)

