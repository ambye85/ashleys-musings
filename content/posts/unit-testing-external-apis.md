---
title: "Unit testing external API's"
date: 2018-12-12T12:00:00+01:00
draft: false
---
I’m currently working on a project to implement a HTTP server. Now, you might be wondering why, given that there are already several great HTTP server implementations, such as Apache and nginx? I have no plans to release this project, but it’s a fantastic project to learn about sockets and the difficulties in testing hard to test aspects of a system.

When I first started learning about unit testing, and specifically about mocking, one of the rules I came across was “don’t mock what you don’t own”. This rule has been well articulated many times over. For example, see [Mockito](https://site.mockito.org/), [Test Double](https://github.com/testdouble/contributing-tests/wiki/Don%27t-mock-what-you-don%27t-own), and [8th Light](https://8thlight.com/blog/eric-smith/2011/10/27/thats-not-yours.html).

When testing a server, the above rule is extremely important. I need to test the behaviour of my HTTP server, especially at the boundaries, which leads to the following question: if I don’t own the socket class and I’m therefore not allowed to mock it, how exactly do I write unit tests for this part of the system?

The recommended approach to testing these hard to test aspects of a system, which are often found at the boundary, is to put a wrapper around the functionality that you don’t own. xUnit patterns is a fantastic resource for learning various testing patterns that can help us to write robust, flexible and decoupled tests. This wrapper approach even has a name — the [Humble Object](http://xunitpatterns.com/Humble%20Object.html).

Why is it important to write flexible, robust and decoupled tests? By doing so, you will develop a system architecture that is flexible, robust and decoupled. This has many benefits, including (but not limited to): independent developabilty and deployability — different teams can work on different aspects of the system concurrently; one implementation can be swapped out for another without any additional changes to dependent modules (or classes/objects) — promotes ease of change; all coupling to external API’s is isolated to one place — if the external API changes, our code only needs to change in one place. By writing code in this way, we make it easy for our employers/clients to request changes and we make it easy for ourselves to implement those changes. As Robert Martin says in [Clean Architecture](https://smile.amazon.co.uk/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164/ref=sr_1_1?ie=UTF8&qid=1544518160&sr=8-1&keywords=clean+architecture), developing software in this manner means that the difficulty of making a change is determined by the scope of the change and not by the shape of the change.

With this in mind, I was curious as to how other people had solved this problem. So, I took to Google and searched for “how to unit test java sockets” and “java mocking sockets”. The top result for each search were the following questions and answers on StackOverflow, respectively: [Testing Java Sockets](https://stackoverflow.com/questions/5577274/testing-java-sockets) and [Mocking a Server-Client Connection with Mockito](https://stackoverflow.com/questions/40414930/mocking-a-server-client-connection-with-mockito).

I was surprised to see that the accepted answers for each promoted mocking the Java Socket class. Indeed, each of these accepted answers had also received numerous upvotes. Given that we don’t own that class (unless you’re developing the JDK), I was further surprised that nobody had raised an objection to this approach or provided a more robust solution using the Humble Object pattern.

Now, I have nothing against people trying to help out other people and provide advice to each other. Knowledge sharing is a fantastic way to improve our (large) community. However, it saddens me to see that the knowledge that is being shared is sometimes flawed, as I believe it to be with the above examples. Instead, I think that a better way to approach mocking sockets (or any other external class/API) is as follows — note that the examples are for an echo server, not a HTTP server, but the principle is the same.

1. Create an interface as a wrapper around the external API:

```java
public interface SocketWrapper {
  void createAndListen(int port);
  String receiveData();
  void sendData(String data);
  void close();
}
```

2. Implement a concrete wrapper with the minimum amount of logic necessary to achieve the desired functionality:

```java
public class ServerSocketWrapper implements SocketWrapper {
  private ServerSocket serverSocket;
  private Socket socket;
  private BufferedReader input;
  private PrintWriter output;

  public void createAndListen(int port) {
    try {
      serverSocket = new ServerSocket(port);
      socket = serverSocket.accept();
      input = new BufferedReader(
          new InputStreamReader(socket.getInputStream()));
      output = new PrintWriter(socket.getOutputStream(), true);
    } catch (UnknownHostException | IOException e) {
      throw new SocketConnectionException();
    }
  }

  public String receiveData() {
    try {
      return input.readLine();
    } catch (IOException e) {
      throw new SocketReadingException();
    }
  }

  public void sendData(String data) {
    output.println(data);
  }

  public void close() {
    try {
      output.close();
      input.close();
      socket.close();
      serverSocket.close();
    } catch (IOException e) {
      throw new SocketClosingException();
    }
  }
}
```

You’ll notice that I’ve wrapped each of the calls I’d like to make to the `ServerSocket` and `Socket` API’s with my own methods. I’ve decided to catch any exceptions and throw my own — there are several reasons for this: `IOException` is a checked exception and I don’t want to re-throw it and also so that I can write a `ServerSocketWrapperStub` which throws each of my exceptions and make assertions about how they will be handled.

It’s important to note that the `ServerSocketWrapper` class will not be unit tested. This particular bit of functionality should therefore be included in an integration test suite.

3. Create mock `SocketWrapper`'s:

```java
public class ServerSocketWrapperSpy implements SocketWrapper {
  private BufferedReader input;
  private PrintWriter output;
  private boolean createAndListenCalled = false;
  private String sentData;
  private boolean closeCalled = false;

  public ServerSocketWrapperSpy(
      BufferedReader input, PrintWriter output) {
    this.input = input;
    this.output = output;
  }

  public void createAndListen(int port) {
    createAndListenCalled = true;
  }

  public String receiveData() {
    try {
      return input.readLine();
    } catch (IOException e) {
      System.err.println("Error reading mock socket input");
    }
  }

  public void sendData(String data) {
    sentData = output.println(data);
  }

  public void close() {
    closeCalled = true;
  }

  public boolean wasCreateAndListenCalled() {
    return createAndListenCalled;
  }

  public String getSentData() {
    return sentData;
  }

  public boolean wasCloseCalled() {
    return closeCalled;
  }
}
```

Here, I’ve created a manual mock but you could easily achieve the same effect by using a mocking framework such as Mockito. Additional mocks would likely include stubs whose methods throw the relevant exceptions in the concrete wrapper.

4. Use the `MockSocketWrapper`'s to unit test the system’s behaviour:

```java
class TestEchoServer {
  private static final int PORT = 1234;

  void testReceivedDataIsEchoedBackInSentData {
    BufferedReader input = new BufferedReader(
        new StringReader("echo\n"));
    PrintWriter output = new PrintWriter(new StringWriter(), true);

    MockServerSocketWrapper socketWrapper =
        new MockServerSocketWrapper(input, output);
    EchoServer echoServer = new EchoServer(socketWrapper);

    echoServer.start(PORT);

    assertTrue(socketWrapper.wasCreateAndListenCalled());
    assertEquals("echo\n", socketWrapper.getSentData());
    assertTrue(socketWrapper.wasCloseCalled());
  }
}
```

By using wrapper objects which wrap around the external API, in this case `ServerSocket` and `Socket`, we have been able to decouple our system’s implementation from the implementation of the socket. If we so wished, we could swap these out for some other network communication facility and our tests and system would be none the wiser. We’ve achieved our aim of architecting a robust, flexible and decoupled system. We’ve given ourselves room to manoeuvre should we need to change the system — the complexity is now about the scope of the change rather than the shape of the change.

By taking this approach, we’ve also adhered to good software design principles. In fact, this approach conforms to all five of the SOLID principles: the wrapper only has one reason to change — Single Responsibility Principle; we can extend the system to use another network facility without touching the `ServerSocketWrapper` — Open/Closed Principle; the `EchoServer` does not know that it is using a derived class — Liskov Substitution Principle; the SocketWrapper only exposes the features of sockets necessary for our application — Interface Segregation Principle; and the `EchoServer` is not dependent on a specific implementation of the `SocketWrapper` — Dependency Inversion Principle.

I hope this article and the included examples have helped to explain why we should not mock external API’s directly. Furthermore, I hope it has helped to explain the many benefits that come from writing tests which are flexible, robust and decoupled from the implementation details.
