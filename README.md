# Web chat
A quick-and-dirty web chat example using socket.io, RabbitMQ, Java and Docker. This illustrates the publish-subscribe 
architectural style with asynchronous messaging in a fully distributed system with two different programming languages.

The system allows N users to chat using their web browsers, but those who use certain swear words are disconnected. 

# Requirements

You need to have installed a Java Virtual Machine, [Node.js](https://nodejs.org/en/) and Docker.

I have only tested this with Oracle Java 1.8 and Docker version 1.13.0 on macOS Sierra 10.12.3, but it will work with other versions of these tools and in most operating systems.

# Install
Clone this repository: `$ git clone https://github.com/fjlopez/web-chat.git`

# Run

To run the RabbitMQ server, open a terminal and run:

```
$ docker run -it --name some-rabbit rabbitmq:3
```

To run the chat server, open a terminal and run:

```
$ cd web-chat-server
$ docker build -t web-chat-server ..
$ docker run -p 3000:3000 -it --link some-rabbit --name web-chat-server -e CLOUDAMQP_URL="amqp://some-rabbit" -t web-chat-server
```

To run the chat clients, open a web browser and point it to the machine where the chat server is running, at port 3000. For example, if the web browser and the server are on the same machine, the URL will be <http://localhost:3000>. You can open as many tabs/windows as you want.

At that point you can start to chat, but the text parser which is looking for swear words is not yet running.

To run the text parser open another terminal and run:

```
$ cd web-chat-processor
$ gradle buildDocker
$ docker run -it --link some-rabbit --name web-chat-processor -e CLOUDAMQP_URL="amqp://some-rabbit" -t web-chat-processor
```

Now you can test the whole system. You can chat normally, but if you use one of the swear words that the text parser recognizes (e.g. "shit") you will be disconnected from the chat.

The system can be fully distributed. You can run the chat server, the chat clients, the AMQP broker and the text parser all in different machines. 

# Improvements
Besides improving the code, which is neither robust nor pretty, there are a few things that you can try to improve the functionality of the system. Some examples:

- You can add other text parsers, which look for other words and run on their own machines. You will need to think about the type of exchange that you need to use in RabbitMQ for all the messages in a queue to reach several different parsers. Some of these parsers can be written in a third programming language, using the proper RabbitMQ libraries.
- You could prevent swear words from reaching the other users of the chat. You will need to the design the system so the text parser must send an explicit "OK" for every message before it can be sent to the other chat users.
- If you are a front-end person, make the GUI responsive. Currently it works fine on mobile browsers (barely tested on my own device Android/Firefox), but the GUI is difficult to use in small screens.

# Credits
The code is based on the examples in <http://socket.io/get-started/chat/>, <https://www.cloudamqp.com/docs/nodejs.html>, <https://github.com/squaremo/amqp.node> and <https://www.rabbitmq.com/tutorials/tutorial-one-java.html>.
