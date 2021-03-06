---
type: mina
title: Chapter 14 - State Machine
navPrev: ../ch13-debugging/ch13-debugging.html
navPrevText: Chapter 13 - Debugging
navUp: ../user-guide-toc.html
navUpText: User Guide
navNext: ../ch15-proxy/ch15-proxy.html
navNextText: Chapter 15 - Proxy
---

# Chapter 14 - State Machine

If you are using MINA to develop an application with complex network interactions you may at some point find yourself reaching for the good old [State pattern](http://home.earthlink.net/~huston2/dp/state.html) to try to sort out some of that complexity. However, before you do that you might want to checkout mina-statemachine which tries to address some of the shortcomings of the State pattern.

{{% toc %}}

## A simple example

Let's demonstrate how mina-statemachine works with a simple example. The picture below shows a state machine for a typical tape deck. The ellipsis are the states while the arrows are the transitions. Each transition is labeled with an event name which triggers that transition.

![](/assets/img/mina/state-diagram.png)

Initially, the tape deck is in the __Empty__ state. When a tape is inserted the __load__ event is fired and the tape deck moves to the __Loaded__ state. In __Loaded__ the __eject__ event will trigger a move back to __Empty__ while the __play__ event will trigger a move to the __Playing__ state. And so on... I think you can work out the rest on your own.

Now let's write some code. The outside world (the code interfacing with the tape deck) will only see the TapeDeck interface:

```java
public interface TapeDeck {
    void load(String nameOfTape);
    void eject();
    void start();
    void pause();
    void stop();
}
```

Next we will write the class which contains the actual code executed when a transition occurs in the state machine. First we will define the states. The states are all defined as constant String objects and are annotated using the @State annotation:

```java
public class TapeDeckHandler {
    @State public static final String EMPTY   = "Empty";
    @State public static final String LOADED  = "Loaded";
    @State public static final String PLAYING = "Playing";
    @State public static final String PAUSED  = "Paused";
}
```

Now when we have the states defined we can set up the code corresponding to each transition. Each transition will correspond to a method in TapeDeckHandler. Each transition method is annotated using the @Transition annotation which defines the event id which triggers the transition (on), the start state of the transition (in) and the end state of the transition (next):

```java
public class TapeDeckHandler {
    @State public static final String EMPTY = "Empty";
    @State public static final String LOADED = "Loaded";
    @State public static final String PLAYING = "Playing";
    @State public static final String PAUSED = "Paused";

    @Transition(on = "load", in = EMPTY, next = LOADED)
    public void loadTape(String nameOfTape) {
        System.out.println("Tape '" + nameOfTape + "' loaded");
    }

    @Transitions({
        @Transition(on = "play", in = LOADED, next = PLAYING),
        @Transition(on = "play", in = PAUSED, next = PLAYING)
    })
    public void playTape() {
        System.out.println("Playing tape");
    }

    @Transition(on = "pause", in = PLAYING, next = PAUSED)
    public void pauseTape() {
        System.out.println("Tape paused");
    }

    @Transition(on = "stop", in = PLAYING, next = LOADED)
    public void stopTape() {
        System.out.println("Tape stopped");
    }

    @Transition(on = "eject", in = LOADED, next = EMPTY)
    public void ejectTape() {
        System.out.println("Tape ejected");
    }
}
```

Please note that the TapeDeckHandler class does not implement the TapeDeck interface. That's intentional.

Now, let's have a closer look at some of this code. The @Transition annotation on the loadTape method

```java
@Transition(on = "load", in = EMPTY, next = LOADED)
public void loadTape(String nameOfTape) {
```

specifies that when the tape deck is in the EMPTY state and the load event occurs the loadTape method will be invoked and then the tape deck will move on to the LOADED state. The @Transition annotations on the pauseTape, stopTape and ejectTape methods should not require any further explanation. The annotation on the playTape method looks slightly different though. As can be seen in the diagram above, when the tape deck is in either the LOADED or in the PAUSED state the play event will play the tape. To have the same method called for multiple transitions the @Transitions annotation has to be used:

```java
@Transitions({
    @Transition(on = "play", in = LOADED, next = PLAYING),
       @Transition(on = "play", in = PAUSED, next = PLAYING)
})
public void playTape() {
```

The @Transitions annotation simply lists multiple transitions for which the annotated method will be called.

<div class="info" markdown="1">
    <strong>More about the @Transition parameters</strong><br>
    <ul>
        <li>If you omit the <tt>on</tt> parameter it will default to &quot;&#42;&quot; which will match any event.</li>
        <li>If you omit the <tt>next</tt> parameter it will default to &quot;_<em>self</em>_&quot; which is an alias for the current state. To create a loop transition in your state machine all you have to do is to omit the <tt>next</tt> parameter.</li>
        <li>The <tt>weight</tt> parameter can be used to define in what order transitions will be searched. Transitions for a particular state will be ordered in ascending order according to their <tt>weight</tt> value. <tt>weight</tt> is 0 by default.</li>
    </ul>
</div>

Now the final step is to create a StateMachine object from the annotated class and use it to create a proxy object which implements TapeDeck:

```java
public static void main(String[] args) {
    TapeDeckHandler handler = new TapeDeckHandler();
    StateMachine sm = StateMachineFactory.getInstance(Transition.class).create(TapeDeckHandler.EMPTY, handler);
    TapeDeck deck = new StateMachineProxyBuilder().create(TapeDeck.class, sm);

    deck.load("The Knife - Silent Shout");
    deck.play();
    deck.pause();
    deck.play();
    deck.stop();
    deck.eject();
}
```

The lines

```java
TapeDeckHandler handler = new TapeDeckHandler();
StateMachine sm = StateMachineFactory.getInstance(Transition.class).create(TapeDeckHandler.EMPTY, handler);
```

creates the StateMachine instance from an instance of TapeDeckHandler. The Transition.class in the call to StateMachineFactory.getInstance(...) tells the factory that we've used the @Transition annotation to build the state machine. We specify EMPTY as the start state. A StateMachine is basically a directed graph. State objects correspond to nodes in the graph while Transition objects correspond to edges. Each @Transition annotation we used in the TapeDeckHandler will correspond to a Transition instance.

<div class="info" markdown="1">
    <strong>Uhhm, what's the difference between @Transition and Transition?</strong><br>
    <tt>@Transition</tt> is the annotation you use to mark a method which should be used when a transition between states occur. Behind the scenes <tt>mina-statemachine</tt> will create instances of the <tt>MethodTransition</tt> class for each <tt>@Transition</tt> annotated method. <tt>MethodTransition</tt> implements the <tt>Transition</tt> interface. As a <tt>mina-statemachine</tt> user you will never use the <tt>Transition</tt> or <tt>MethodTransition</tt> types directly.
</div>

The TapeDeck instance is created by calling StateMachineProxyBuilder:

```java
TapeDeck deck = new StateMachineProxyBuilder().create(TapeDeck.class, sm);
```

The StateMachineProxyBuilder.create() method takes the interfaces the returned proxy object should implement and the StateMachine instance which will receive the events generated by the method calls on the proxy.

When the code is executed the output should be:

```text
Tape 'The Knife - Silent Shout' loaded
Playing tape
Tape paused
Playing tape
Tape stopped
Tape ejected
```

<div class="info" markdown="1">
    <strong>What does all this have to do with MINA?</strong><br>
    As you might have noticed there's nothing MINA specific about this example. But don't be alarmed. Later on we will see how to create state machines for MINA's <tt>IoHandler</tt> interface.
</div>

# How does it work?

Let's walk through what happens when a method is called on the proxy.

## Lookup a StateContext object

The StateContext object is important because it holds the current State. When a method is called on the proxy it will ask a StateContextLookup instance to get the StateContext from the method's arguments. Normally, the StateContextLookup implementation will loop through the method arguments and look for a particular type of object and use it to retrieve a StateContext object. If no StateContext has been assigned yet the StateContextLookup will create one and store it in the object.

When proxying MINA's IoHandler we will use a IoSessionStateContextLookup instance which looks for an IoSession in the method arguments. It will use the IoSession's attributes to store a separate instance of StateContext for each MINA session. That way the same state machine can be used for all MINA sessions without them interfering with each other.

<div class="note" markdown="1">
    In the example above we never specified what <tt>StateContextLookup</tt> implementation to use when we created the proxy using <tt>StateMachineProxyBuilder</tt>. If not specified a <tt>SingletonStateContextLookup</tt> will be used. <tt>SingletonStateContextLookup</tt> totally disregards the method arguments passed to it &ndash; it'll always return the same <tt>StateContext</tt> object. Obviously this won't be very useful when the same state machine is used concurrently by many clients as will be the case when we proxy <tt>IoHandler</tt> later on.
</div>

## Convert the method invocation into an Event object

All method invocations on the proxy object will be translated into Event objects by the proxy. An Event has an id and zero or more arguments. The id corresponds to the name of the method and the event arguments correspond to the method arguments. The method call deck.load("The Knife - Silent Shout") corresponds to the event {id = "load", arguments = ["The Knife - Silent Shout"]}. The Event object also contains a reference to the StateContext object looked up previously.

## Invoke the StateMachine

Once the Event object has been created the proxy will call StateMachine.handle(Event). StateMachine.handle(Event) loops through the Transition objects of the current State in search for a Transition instance which accepts the current Event. This process will stop after a Transition has been found. The Transition objects will be searched in order of weight (typically specified by the @Transition annotation).

## Execute the Transition

The final step is to call Transition.execute(Event) on the Transition which matched the Event. After the Transition has been executed the StateMachine will update the current State with the end state defined by the Transition.

<div class="info" markdown="1">
    <tt>Transition</tt> is an interface. Every time you use the <tt>@Transition</tt> annotation a <tt>MethodTransition</tt> object will be created.
</div>

# MethodTransition

MethodTransition is very important and requires some further explanation. MethodTransition matches an Event if the event's id matches the on parameter of the @Transition annotation and the annotated method's arguments are assignment compatible with a subset of the event's arguments.

So, if the Event looks like {id = "foo", arguments = [a, b, c]} the method

```java
@Transition(on = "foo")
public void someMethod(One one, Two two, Three three) { ... }
```

matches if and only if ((a instanceof One && b instanceof Two && c instanceof Three) == true). On match the method will be called with the matching event arguments bound to the method's arguments:

```java
someMethod(a, b, c);
```

<div class="info" markdown="1">
    <tt>Integer</tt>, <tt>Double</tt>, <tt>Float</tt>, etc also match their primitive counterparts <tt>int</tt>, <tt>double</tt>, <tt>float</tt>, etc.
</div>

As stated above also a subset would match:

```java
@Transition(on = "foo")
public void someMethod(Two two) { ... }
```

matches if ((a instanceof Two || b instanceof Two || c instanceof Two) == true). In this case the first matching event argument will be bound to the method argument named two when someMethod is called.

A method which takes no arguments always matches if the event id matches:

```java
@Transition(on = "foo")
public void someMethod() { ... }
```

To make things even more complicated the first two method arguments also matches against the Event class and the StateContext interface. This means that

```java
@Transition(on = "foo")
public void someMethod(Event event, StateContext context, One one, Two two, Three three) { ... }
@Transition(on = "foo")
public void someMethod(Event event, One one, Two two, Three three) { ... }
@Transition(on = "foo")
public void someMethod(StateContext context, One one, Two two, Three three) { ... }
```

also matches the Event {id = "foo", arguments = [a, b, c]} if ((a instanceof One && b instanceof Two && c instanceof Three) == true). The current Event object will be bound to the event method argument and the current StateContext will be bound to context when someMethod is invoked.

As before a subset of the event arguments can be used. Also, a specific StateContext implementation may be specified instead of using the generic interface:

```java
@Transition(on = "foo")
public void someMethod(MyStateContext context, Two two) { ... }
```

<div class="note" markdown="1">
    The order of the method arguments is important. If the method needs access to the current <tt>Event</tt> it must be specified as the first method argument. <tt>StateContext</tt> has to be the either the second arguments if the first is <tt>Event</tt> or the first argument. The event arguments also have to match in the correct order. <tt>MethodTransition</tt> will not try to reorder the event's arguments in search for a match.
</div>

If you've made it this far, congratulations! I realize that the section above might be a little hard to digest. Hopefully some examples could make things clearer:

Consider the Event {id = "messageReceived", arguments = [ArrayList a = [...], Integer b = 1024]}. The following methods match this Event:

```java
// All method arguments matches all event arguments directly
@Transition(on = "messageReceived")
public void messageReceived(ArrayList l, Integer i) { ... }

// Matches since ((a instanceof List && b instanceof Number) == true)
@Transition(on = "messageReceived")
public void messageReceived(List l, Number n) { ... }

// Matches since ((b instanceof Number) == true)
@Transition(on = "messageReceived")
public void messageReceived(Number n) { ... }

// Methods with no arguments always matches
@Transition(on = "messageReceived")
public void messageReceived() { ... }

// Methods only interested in the current Event or StateContext always matches
@Transition(on = "messageReceived")
public void messageReceived(StateContext context) { ... }

// Matches since ((a instanceof Collection) == true)
@Transition(on = "messageReceived")
public void messageReceived(Event event, Collection c) { ... }
```

The following would not match:

```java
// Incorrect ordering
@Transition(on = "messageReceived")
public void messageReceived(Integer i, List l) { ... }

// ((a instanceof LinkedList) == false)
@Transition(on = "messageReceived")
public void messageReceived(LinkedList l, Number n) { ... }

// Event must be first argument
@Transition(on = "messageReceived")
public void messageReceived(ArrayList l, Event event) { ... }

// StateContext must be second argument if Event is used
@Transition(on = "messageReceived")
public void messageReceived(Event event, ArrayList l, StateContext context) { ... }

// Event must come before StateContext
@Transition(on = "messageReceived")
public void messageReceived(StateContext context, Event event) { ... }
```

## State inheritance

State instances may have a parent State. If StateMachine.handle(Event) cannot find a Transition matching the current Event in the current State it will search the parent State. If no match is found there either the parent's parent will be searched and so on.

This feature is useful when you want to add some generic code to all states without having to specify @Transition annotations for each state. Here's how you create a hierarchy of states using the @State annotation:

```java
@State    public static final String A = "A";
@State(A) public static final String B = "A->B";
@State(A) public static final String C = "A->C";
@State(B) public static final String D = "A->B->D";
@State(C) public static final String E = "A->C->E";
```

## Error handling using state inheritance

Let's go back to the TapeDeck example. What happens if you call deck.play() when there's no tape in the deck? Let's try:

```java
public static void main(String[] args) {
    ...
    deck.load("The Knife - Silent Shout");
    deck.play();
    deck.pause();
    deck.play();
    deck.stop();
    deck.eject();
    deck.play();
}

...
Tape stopped
Tape ejected
Exception in thread "main" o.a.m.sm.event.UnhandledEventException: 
Unhandled event: org.apache.mina.statemachine.event.Event@15eb0a9[id=play,...]
    at org.apache.mina.statemachine.StateMachine.handle(StateMachine.java:285)
    at org.apache.mina.statemachine.StateMachine.processEvents(StateMachine.java:142)
    ...
```

Oops! We get an UnhandledEventException because when we're in the Empty state there's no transition which handles the play event. We could add a special transition to all states which handles unmatched Event objects:

```java
@Transitions({
    @Transition(on = "*", in = EMPTY, weight = 100),
    @Transition(on = "*", in = LOADED, weight = 100),
    @Transition(on = "*", in = PLAYING, weight = 100),
    @Transition(on = "*", in = PAUSED, weight = 100)
})
public void error(Event event) {
    System.out.println("Cannot '" + event.getId() + "' at this time");
}
```

Now when you run the main() method above you won't get an exception. The output should be:

```text
    ...
    Tape stopped
    Tape ejected
    Cannot 'play' at this time.
```

Now this seems to work very well, right? But what if we had 30 states instead of only 4? Then we would need 30 @Transition annotations on the error() method. Not good. Let's use state inheritance instead:

```java
public static class TapeDeckHandler {
    @State public static final String ROOT = "Root";
    @State(ROOT) public static final String EMPTY = "Empty";
    @State(ROOT) public static final String LOADED = "Loaded";
    @State(ROOT) public static final String PLAYING = "Playing";
    @State(ROOT) public static final String PAUSED = "Paused";
    
    ...
    
    @Transition(on = "*", in = ROOT)
    public void error(Event event) {
        System.out.println("Cannot '" + event.getId() + "' at this time");
    }
}
```

The result will be the same but things will be much easier to maintain with this last approach.

## mina-statemachine with IoHandler

Now we're going to convert our tape deck into a TCP server and extend it with some more functionality. The server will receive commands like load <tape>, play, stop, etc. The responses will either be positive + <message> or negative - <message>. The protocol is text based, all commands and responses are lines of UTF-8 text terminated by CRLF (i.e. \r\n in Java). Here's an example session:

```text
telnet localhost 12345
S: + Greetings from your tape deck!
C: list
S: + (1: "The Knife - Silent Shout", 2: "Kings of convenience - Riot on an empty street")
C: load 1
S: + "The Knife - Silent Shout" loaded
C: play
S: + Playing "The Knife - Silent Shout"
C: pause
S: + "The Knife - Silent Shout" paused
C: play
S: + Playing "The Knife - Silent Shout"
C: info
S: + Tape deck is playing. Current tape: "The Knife - Silent Shout"
C: eject
S: - Cannot eject while playing
C: stop
S: + "The Knife - Silent Shout" stopped
C: eject
S: + "The Knife - Silent Shout" ejected
C: quit
S: + Bye! Please come back!
```

The complete code for the TapeDeckServer described in this section is available in the org.apache.mina.example.tapedeck package in the mina-example module in the Subversion repository. The code uses a MINA ProtocolCodecFilter to convert bytes from/to Command objects. There is one Command implementation for each type of request the server recognizes. We will not describe the codec implementation here in any detail.

Now, let's have a look at how this server works. The important class which implements the state machine is the TapeDeckServer class. The first thing we do is to define the states:

```java
@State public static final String ROOT = "Root";
@State(ROOT) public static final String EMPTY = "Empty";
@State(ROOT) public static final String LOADED = "Loaded";
@State(ROOT) public static final String PLAYING = "Playing";
@State(ROOT) public static final String PAUSED = "Paused";
```
 
Nothing new there. However, the methods which handle the events now look different. Let's look at the playTape method:

```java
@IoHandlerTransitions({
    @IoHandlerTransition(on = MESSAGE_RECEIVED, in = LOADED, next = PLAYING),
    @IoHandlerTransition(on = MESSAGE_RECEIVED, in = PAUSED, next = PLAYING)
})
public void playTape(TapeDeckContext context, IoSession session, PlayCommand cmd) {
    session.write("+ Playing \"" + context.tapeName + "\"");
}
```

This code doesn't use the general @Transition and @Transitions annotations used previously but rather the MINA specific @IoHandlerTransition and @IoHandlerTransitions annotations. This are preferred when creating state machines for MINA's IoHandler interface as they let you use a Java enum for the event ids instead of strings as we used before. There are also corresponding annotations for MINA's IoFilter interface.

We're now using MESSAGE_RECEIVED instead of "play" for the event name (the on attribute in @IoHandlerTransition). This constant is defined in org.apache.mina.statemachine.event.IoHandlerEvents and has the value "messageReceived" which of course corresponds to the messageReceived() method in MINA's IoHandler interface. Thanks to Java5's static imports we don't have to write out the name of the class holding the constant. We just need to put the

```java
import static org.apache.mina.statemachine.event.IoHandlerEvents.*;
```

statement in the imports section.

Another thing that has changed is that we're using a custom StateContext implementation, TapeDeckContext. This class is used to keep track of the name of the current tape:

```java
static class TapeDeckContext extends AbstractStateContext {
    public String tapeName;
}
```

<div class="info" markdown="1">
    <strong>Why not store tape name in IoSession?</strong><br>
    We could have stored the name of the tape as an attribute in the <tt>IoSession</tt> but using a custom <tt>StateContext</tt> is recommended since it provides type safety.
</div>

The last thing to note about the playTape() method is that it takes a PlayCommand as its last argument. The last argument corresponds to the message argument of IoHandler's messageReceived(IoSession session, Object message) method. This means that playTape() method will only be called if the bytes sent by the client can be decoded as a PlayCommand.

Before the tape deck can play anything a tape has to be loaded. When a LoadCommand is received from the client the supplied tape number will be used to get the name of the tape to load from the tapes array of available tapes:

```java
@IoHandlerTransition(on = MESSAGE_RECEIVED, in = EMPTY, next = LOADED)
public void loadTape(TapeDeckContext context, IoSession session, LoadCommand cmd) {
    if (cmd.getTapeNumber() < 1 || cmd.getTapeNumber() > tapes.length) {
        session.write("- Unknown tape number: " + cmd.getTapeNumber());
        StateControl.breakAndGotoNext(EMPTY);
    } else {
        context.tapeName = tapes[cmd.getTapeNumber() - 1];
        session.write("+ \"" + context.tapeName + "\" loaded");
    }
}
```

This code uses the StateControl class to override the next state. If the user specify an unknown tape number we shouldn't move to the LOADED state but instead remain in EMPTY which is what the

```java
StateControl.breakAndGotoNext(EMPTY);
```

line does. The StateControl class is described more in a later section.

The connect() method will always be called at the start of a session when MINA calls sessionOpened() on the IoHandler:

```java
@IoHandlerTransition(on = SESSION_OPENED, in = EMPTY)
public void connect(IoSession session) {
    session.write("+ Greetings from your tape deck!");
}
```

All it does is to write the greeting to the client. The state machine will remain in the EMPTY state.

The pauseTape(), stopTape() and ejectTape() methods are very similar to playTape() and won't be described in any detail. The listTapes(), info() and quit() methods should be simple enough to understand by now, too. Please note how these last three methods are used for the ROOT state. This means that the list, info and quit commands can be issued in any state.

Now let's have a look at error handling. The error() method will be called when the client sends a Command which isn't legal in the current state:

```java
@IoHandlerTransition(on = MESSAGE_RECEIVED, in = ROOT, weight = 10)
public void error(Event event, StateContext context, IoSession session, Command cmd) {
    session.write("- Cannot " + cmd.getName() + " while " 
           + context.getCurrentState().getId().toLowerCase());
}
```

error() has been given a higher weight than listTapes(), info() and quit() to prevent it to be called for any of those commands. Notice how error() uses the StateContext object to get hold of the id of the current state. The values of the String constants which are annotated with the @State annotation (Empty, Loaded etc) will be used by mina-statemachine as state id.

The commandSyntaxError() method will be called when a CommandSyntaxException has been thrown by our ProtocolDecoder. It simply prints out that the line sent by the client couldn't be converted into a Command.

The exceptionCaught() will be called for any thrown exception except CommandSyntaxException (it has a higher weight than the commandSyntaxError() method). It closes the session immediately.

The last @IoHandlerTransition method is unhandledEvent() which will be called if none of the other @IoHandlerTransition methods match the Event. We need this since we don't have @IoHandlerTransition annotations for all possible types of events in all states (e.g., we never handle messageSent events). Without this mina-statemachine throws an exception if an Event is handled by the state machine.

The last piece of code we're going to have a look at is the code which creates the IoHandler proxy and the main() method:

```java
private static IoHandler createIoHandler() {
    StateMachine sm = StateMachineFactory.getInstance(IoHandlerTransition.class).create(EMPTY, new TapeDeckServer());
        
    return new StateMachineProxyBuilder().setStateContextLookup(
            new IoSessionStateContextLookup(new StateContextFactory() {
                public StateContext create() {
                    return new TapeDeckContext();
                }
            })).create(IoHandler.class, sm);
}

// This code will work with MINA 1.0/1.1:
public static void main(String[] args) throws Exception {
    SocketAcceptor acceptor = new SocketAcceptor();
    SocketAcceptorConfig config = new SocketAcceptorConfig();
    config.setReuseAddress(true);
    ProtocolCodecFilter pcf = new ProtocolCodecFilter(
            new TextLineEncoder(), new CommandDecoder());
    config.getFilterChain().addLast("codec", pcf);
    acceptor.bind(new InetSocketAddress(12345), createIoHandler(), config);
}

// This code will work with MINA trunk:
public static void main(String[] args) throws Exception {
    SocketAcceptor acceptor = new NioSocketAcceptor();
    acceptor.setReuseAddress(true);
    ProtocolCodecFilter pcf = new ProtocolCodecFilter(
            new TextLineEncoder(), new CommandDecoder());
    acceptor.getFilterChain().addLast("codec", pcf);
    acceptor.setHandler(createIoHandler());
    acceptor.setLocalAddress(new InetSocketAddress(PORT));
    acceptor.bind();
}
```

createIoHandler() creates a StateMachine just like we did before except that we specify IoHandlerTransition.class instead of Transition.class in the call to StateMachineFactory.getInstance(...). This is necessary since we're now using the @IoHandlerTransition annotation. Also, this time we use IoSessionStateContextLookup and a custom StateContextFactory when we create the IoHandler proxy. If we didn't use IoSessionStateContextLookup all clients would share the same state machine which isn't desirable.

The main() method creates the SocketAcceptor and attaches a ProtocolCodecFilter which decodes/encodes Command objects to its filter chain. Finally, it binds to port 12345 using an IoHandler instance created by the createIoHandler() method.

# Advanced topics

## Changing state programmatically

To be written...

## Calling the state machine recursively

To be written...
