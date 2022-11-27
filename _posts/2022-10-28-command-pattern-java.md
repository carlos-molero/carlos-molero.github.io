---
layout: post
title: The Command Pattern In Java
author: Carlos Molero Mata
description: A simple introduction to the command pattern in the java programming language with a simple example.
header-style: text
tags: [java, design patterns]
---

Learning design patterns is fundamental if you want to build robust real world applications. This time, we are going to learn about the command pattern, how to implement it in Java and when it can be useful.

## Characteristics of the command pattern

A quick search on the internet will tell us this about the command pattern:

> In object-oriented programming, the command pattern is a behavioral design pattern in which an object is used to encapsulate all information needed to perform an action or trigger an event at a later time. This information includes the method name, the object that owns the method and values for the method parameters.

We can extract some conclusions about the command pattern from this paraphraph:

- It is used in Object Oriented Programming (OOP).
- It is a behavioral design pattern, these are design patterns that identify common communication patterns among objects. By doing so, these patterns increase flexibility in carrying out communication.
- It is an event-based pattern.
- The command is the fundamental piece of the pattern and it contains all the information to fullfill its purpose.

The general command pattern structure is comprised of 5 components. The names may vary depending on the person, but they serve their purpose.

![command-pattern-java](/img/posts/command-pattern-java.webp)
_Image from [OpenGenus](https://iq.opengenus.org/command-pattern-in-java/)_

1. **Command Interface**: provides an interface that all concrete command classes will use.
2. **Concrete Command classes**: calls a specific method in the Receiver class.
3. **Receiver**: the class that does all the work behind the scenes where all the inner details are.
4. **Invoker**: the class that calls the Command objects's execute function when needed to perform the request.
5. **Client**: the client that can parameterize and pick what commands they want to request.

## When is it useful?

The command pattern is generally **useful when your application needs to keep a history of executed actions and be able to undo them if necessary**.

Think of an application like Word office, in this document editor we can identify multiple commands such as making a sentence bold, italic, delete a row from a table etc... Also, all these commands can be undone if necessary because the application saves the history of executions.

## An example

We are going to create a simple example to illustrate the command pattern, we will simulate an application that has the functionality of switching a light bulb on and off.

### Receiver

The receiver is the light bulb, the class which contains the object that we want to manipulate and the methods to do so.

```java
public class LightBulb {
    private boolean on = false;

    public void switchOn() {
        this.on = true;
        System.out.println("Light is ON");
    }

    public void switchOff() {
        this.on = false;
        System.out.println("Light is OFF");
    }
}
```

### Command interface

The command interface will declare methods to be implemented in all the commands. All of them will have to implement the `execute()` and `undo()` methods.

```java
public interface Command {
    // Just for demonstration purposes
    String getName();
    void execute();
    void undo();
}
```

### Invoker

The invoker is the class which controls the execution of the commands and also stores the history of executions in case the user wants to rollback. We store the history of executions in a `Deque` which is short for _double ended queue_, [read more about it here](https://docs.oracle.com/javase/7/docs/api/java/util/Deque.html) if you're not familiar with this data structure.

We will store a maximum of 10 commands, whenever the user "clicks" the undo button we will get the last one, call the `undo()` method and remove it from the queue.

```java
public class LightSwitch {
    private Command command;
    private Deque<Command> queue = new ArrayDeque<>();

    public void setCommand(Command command) {
        this.command = command;
    }

    public void pressCommand() {
        System.out.println("[EXECUTE]: " + command.getName());
        this.command.execute();
        if (queue.size() > 9) {
            queue.pop();
        }
        queue.add(command);
        System.out.println("[ADDED TO HISTORY]: " + command.getName());
    }

    public void undoLastCommand() {
        Command command = queue.removeLast();
        System.out.println("[UNDO]: " + command.getName());
        command.undo();
    }
}
```

### Concrete commands

These classes will manipulate the state of the light bulb and will revert it to its previous state when `undo()` is called.

```java
public class SwitchOnCommand implements Command {
    LightBulb lightBulb;

    public SwitchOnCommand(LightBulb lightBulb) {
        this.lightBulb = lightBulb;
    }

    @Override
    public String getName() {
        return "SwitchOnCommand";
    }

    @Override
    public void execute() {
        lightBulb.switchOn();
    }

    @Override
    public void undo() {
        lightBulb.switchOff();
    }
}
```

```java
public class SwitchOffCommand implements Command {
    LightBulb lightBulb;

    public SwitchOffCommand(LightBulb lightBulb) {
        this.lightBulb = lightBulb;
    }

    @Override
    public String getName() {
        return "SwitchOffCommand";
    }

    @Override
    public void execute() {
        lightBulb.switchOff();
    }

    @Override
    public void undo() {
        lightBulb.switchOn();
    }
}
```

### Client

For simplicity, the entry point of our app will be used as the client. The client is the class which contains the invoker and use it to execute commands:

```java
public class App {
    public static void main(String[] args) {
        LightBulb lightBulb = new LightBulb();
        LightSwitch lightSwitch = new LightSwitch();

        Command switchOff = new SwitchOffCommand(lightBulb);
        Command switchOn = new SwitchOnCommand(lightBulb);

        lightSwitch.setCommand(switchOn);
        lightSwitch.executeCommand();
        lightSwitch.setCommand(switchOff);
        lightSwitch.executeCommand();

        lightSwitch.undoLastCommand();
        lightSwitch.undoLastCommand();
    }
}
```

If we run this app we can expect the following console output:

```
[EXECUTE]: SwitchOnCommand
Light is ON
[ADDED TO HISTORY]: SwitchOnCommand
[EXECUTE]: SwitchOffCommand
Light is OFF
[ADDED TO HISTORY]: SwitchOffCommand
[UNDO]: SwitchOffCommand
Light is ON
[UNDO]: SwitchOnCommand
Light is OFF
```

That's all for today, I hope it has been clear to you what the command pattern is, when it is convenient to use it and how to implement it in Java.
