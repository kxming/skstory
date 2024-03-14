---
title: 'JavaScript Design Patterns'
date: 2024-02-15T16:22:38+08:00
draft: true
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*xOAzIinBSTLixy37KkW7ag.jpeg"
categories: ["Javascript"]
tags: ["Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "Design patterns can be broadly classified into three types: Creation patterns, Structural patterns, Behavioral patterns"
---

## Preamble

Design patterns can be broadly classified into three types:

**Creation patterns**: Factory Method Pattern, Abstract Factory Pattern, Singleton Pattern, Builder Pattern, Prototype Pattern.

**Structural patterns**: Adapter Pattern, Decorator Pattern, Proxy Pattern, Facade Pattern, Bridge Pattern, Composite Pattern, Flyweight Pattern.

**Behavioral patterns**: Strategy Pattern, Template Method Pattern, Observer Pattern, Iterator Pattern, Chain of Responsibility Pattern, Command Pattern, Memento Pattern, State Pattern, Visitor Pattern, Mediator Pattern, Interpreter Pattern.

---

## Factory Method Pattern

The factory method pattern defines an interface for creating objects, but it’s up to the subclass to decide which class to instantiate. It separates the creation and use of the objects, making the system more flexible. Here is a code example:

```
// Define an abstract class
class Animal {
  speak() {
    throw new Error('This method must be implemented.');
  }
}

// Implement specific classes
class Dog extends Animal {
  speak() {
    return 'Woof!';
  }
}

class Cat extends Animal {
  speak() {
    return 'Meow!';
  }
}

// Implement factory method
class AnimalFactory {
  createAnimal(animalType) {
      switch(animalType) {
        case 'dog':
          return new Dog();
        case 'cat':
          return new Cat();
        default:
          throw new Error(`Invalid animal type: ${animalType}`);
      }
  }
}

// Use factory method to create objects
const animalFactory = new AnimalFactory();
const dog = animalFactory.createAnimal('dog');
console.log(dog.speak()); // Output: Woof!
const cat = animalFactory.createAnimal('cat');
console.log(cat.speak()); // Output: Meow!
```

## Singleton Pattern

The purpose of the Singleton Pattern is to ensure a class has only one instance and to provide a global access point to it. Here’s a code sample:

```
class Logger {
  constructor() {
    if (!Logger.instance) {
      this.logs = [];
      Logger.instance = this;
    }

    return Logger.instance;
  }

  log(message) {
    this.logs.push(message);
    console.log(`Logger: ${message}`);
  }

  printLogCount() {
    console.log(`Number of logs: ${this.logs.length}`);
  }
}

// A global variable can be used to access an instance
const logger = new Logger();
Object.freeze(logger);

// The output should be the same for each instance
logger.log('First message'); // Output: Logger: First message
logger.printLogCount(); // Output: Number of logs: 1

const anotherLogger = new Logger(); // At this point, returns an existing instance
anotherLogger.log('Second message'); // Output: Logger: Second message
anotherLogger.printLogCount(); // Output: Number of logs: 2
```

## Builder Pattern

The Builder Pattern is a design pattern that is used to construct an object step by step and is generally used to construct complex objects. Here’s a code sample:

```
// Create Product class
class Sandwich {
  constructor() {
    this.ingredients = [];
  }

  addIngredient(ingredient) {
    this.ingredients.push(ingredient);
  }

  toString() {
    return this.ingredients.join(', ');
  }
}

// Create a builder class
class SandwichBuilder {
  constructor() {
    this.sandwich = new Sandwich();
  }

  reset() {
    this.sandwich = new Sandwich();
  }

  putMeat(meat) {
    this.sandwich.addIngredient(meat);
  }

  putCheese(cheese) {
    this.sandwich.addIngredient(cheese);
  }

  putVegetables(vegetables) {
    this.sandwich.addIngredient(vegetables);
  }

  get result() {
    return this.sandwich;
  }
}

// Create a builder for user (director) to use
class SandwichMaker {
  constructor() {
    this.builder = new SandwichBuilder();
  }

  createCheeseSteakSandwich() {
    this.builder.reset();
    this.builder.putMeat('ribeye steak');
    this.builder.putCheese('american cheese');
    this.builder.putVegetables(['peppers', 'onions']);
    return this.builder.result;
  }
}

// Build a Sandwich
const sandwichMaker = new SandwichMaker();
const sandwich = sandwichMaker.createCheeseSteakSandwich();
console.log(`Your sandwich: ${sandwich}`); // Output: Your sandwich: ribeye steak, american cheese, peppers, onions
```

## Prototype Pattern

The Prototype Pattern is a creational design pattern, which can be used when the cost of creating objects is relatively expensive but objects with the same properties can be created by cloning. The Prototype Pattern separates the process of creating objects from the process of using these objects, as it creates new objects by cloning existing ones, thus avoiding costly object creation. Because JavaScript natively supports object cloning (i.e., copying), implementing the Prototype Pattern in JavaScript is relatively simple.

Here is a code sample of the Prototype Pattern:

```
// Create a prototype object
const carPrototype = {
  wheels: 4,
  color: 'red',
  start() {
    console.log('Starting the car...');
  },
  stop() {
    console.log('Stopping the car...');
  },
};

// Clone with Object.create() method
const car1 = Object.create(carPrototype);
console.log(car1); // Output: {}

car1.wheels = 6;
console.log(car1.wheels); // Output: 6
console.log(car1.color); // Output: red

car1.start(); // Output: Starting the car...
car1.stop(); // Output: Stopping the car...

// Clone another object
const car2 = Object.create(carPrototype);
console.log(car2); // Output: {}

car2.color = 'blue'; 
console.log(car2.color); // Output: blue
console.log(car2.wheels); // Output: 4

car2.start(); // Output: Starting the car...
car2.stop(); // Output: Stopping the car...
```

In this example, we created a prototype object called carPrototype. Then we cloned this prototype object using the Object.create() method. As we performed a shallow copy, we can modify the object’s properties before using it and car2 and car1 objects’ start() and stop() methods are the same because they come from the same prototype object.

The Prototype Pattern has the advantage of providing a convenient way to create objects with the same properties. It can reduce duplicate code and save time and resources when creating objects. However, it also has some disadvantages. For example, deep copying might cause unexpected problems because all properties are copied and these properties may also refer to other objects.

## Adapter Pattern

The Adapter Pattern is a structural design pattern that allows incompatible objects to be wrapped in an adapter, making them work together. Here’s a code sample of the Adapter Pattern:

```
// Target interface
class Target {
  request() {
    console.log('Target: request has been called');
  }
}

// The class that needs to be adapted
class Adaptee {
  specificRequest() {
    console.log('Adaptee method has been accessed');
  }
}

// Adapter class, transforming Adaptee to Target
class Adapter extends Target {
  constructor(adaptee) {
    super();
    this.adaptee = adaptee;
  }

  request() {
    this.adaptee.specificRequest();
  }
}

// Use adapter to decouple client from Adaptee
const client = new Adapter(new Adaptee());
client.request(); // Output: Adaptee method has been accessed
```

In the code above, we have a target interface Target and a class Adaptee that needs to be adapted. We created an adapter class Adapter to transform Adaptee to Target and used the adapter to communicate with the client, which calls the request() method, thereby implementing the functionality of Adaptee.

## Proxy Pattern

The Proxy Pattern is a structural design pattern that provides a placeholder or proxy for an object when accessing it, thus offering control over the access to the object. Here’s a code sample of the Proxy Pattern:

```
// Subject interface
class Subject {
  request() {
    console.log('Subject: Handling request');
  }
}

// Real Subject class
class RealSubject extends Subject {
  request() {
    console.log('RealSubject: Handling request');
  }
}

// Proxy class
class Proxy extends Subject {
  constructor(realSubject) {
    super();
    this.realSubject = realSubject;
  }

  request() {
    if (this.checkAccess()) {
      this.realSubject.request();
      this.logAccess();
    }
  }

  checkAccess() {
    console.log('Proxy: Checking access rights');
    return true;
  }

  logAccess() {
    console.log('Proxy: Logging access');
  }
}

// Accessing the real object using proxy
const realSubject = new RealSubject();
const proxy = new Proxy(realSubject);

proxy.request();
```

In the code above, we have a subject interface Subject and a real subject class RealSubject. We created a proxy class Proxy that encapsulates a real subject and offers additional functionality when accessing it, such as checking access rights and logging access. By instantiating the RealSubject class and wrapping it in a Proxy class, we access the real subject object through the proxy.

## Facade Pattern

The Facade Pattern is a structural design pattern that provides a simpler interface for a set of complex subsystems. Here’s a code sample of the Facade Pattern:

```
// Subsystem1
class Subsystem1 {
  operation1() {
    console.log('Subsystem1: Perform operation1');
  }
}

// Subsystem2
class Subsystem2 {
  operation2() {
    console.log('Subsystem2: Perform operation2');
  }
}

// Facade class
class Facade {
  constructor() {
    this.subsystem1 = new Subsystem1();
    this.subsystem2 = new Subsystem2();
  }

  operation() {
    this.subsystem1.operation1();
    this.subsystem2.operation2();
  }
}

// Client code
const facade = new Facade();
facade.operation(); // Output: Subsystem1: Perform operation1, Subsystem2: Perform operation2
```

In the code above, we have two subsystems Subsystem1 and Subsystem2, both of which provide complex operations. We created a “Facade” class using the Facade Pattern, which provides a simple interface by combining the operations of Subsystem1 and Subsystem2 objects. Finally, we instantiate the Facade class and call the operation() method to complete the complex functional operation.

## Bridge Pattern

The Bridge Pattern is a structural design pattern that separates an object’s abstraction from its implementation, allowing the two to vary independently. Here’s a code sample of the Bridge Pattern:

```
// Implementor interface
class Implementor {
  operationImpl() {
    console.log('Implementor: Perform operation');
  }
}

// Abstract class
class Abstraction {
  constructor(implementor) {
    this.implementor = implementor;
  }

  operation() {
    this.implementor.operationImpl();
  }
}

// Extended Abstract class
class RefinedAbstraction extends Abstraction {
  otherOperation() {
    console.log('RefinedAbstraction: Other operation');
  }
}

// Utilizing the Bridge Pattern
const implementor = new Implementor();
const abstraction = new Abstraction(implementor);
abstraction.operation(); // Output: Implementor: Perform operation

const refinedAbstraction = new RefinedAbstraction(implementor);
refinedAbstraction.operation(); // Output: Implementor: Perform operation
refinedAbstraction.otherOperation(); // Output: RefinedAbstraction: Other operation
```

In the code above, we have an Implementor interface and an abstract class — Abstraction. By creating an extended abstract class, RefinedAbstraction, to extend the functionality of the abstract class, we are utilizing an instance object of the Implementor interface. We then instantiate Implementor and create two objects with different behaviors by passing the “Implementor” object in the declarations of the Abstraction and RefinedAbstraction classes. By separating the implementation from the abstraction, we can freely combine the implementation with the abstraction, making it easier to extend.

## Composite Pattern

The Composite Pattern is a structural design pattern that uses a tree structure to represent the part-whole hierarchy of objects, allowing users to handle individual objects and combinations of objects in a uniform way. Here’s a code sample of the Composite Pattern:

```
// Abstract Component
class Component {
  constructor(name) {
    this.name = name;
  }

  operation() {
    console.log(`Component ${this.name}: Perform operation`);
  }

  add(component) {
    console.log('Component: Unsupported operation');
  }

  remove(component) {
    console.log('Component: Unsupported operation');
  }

  getChild(index) {
    console.log('Component: Unsupported operation');
  }
}

// Leaf Node
class Leaf extends Component {
  constructor(name) {
    super(name);
  }
}

// Branch Node
class Composite extends Component {
  constructor(name) {
    super(name);
    this.children = [];
  }

  add(component) {
    this.children.push(component);
  }

  remove(component) {
    const index = this.children.indexOf(component);
    if (index >= 0) {
      this.children.splice(index, 1);
    }
  }

  getChild(index) {
    return this.children[index];
  }
}

// Using the Composite Pattern
const root = new Composite('Root');
const branch1 = new Composite('Branch1');
const branch2 = new Composite('Branch2');
const leaf1 = new Leaf('Leaf1');
const leaf2 = new Leaf('Leaf2');
const leaf3 = new Leaf('Leaf3');

root.add(branch1);
root.add(branch2);
branch1.add(leaf1);
branch1.add(leaf2);
branch2.add(leaf3);

root.operation(); // Output: Component Root: Perform operation
branch1.operation(); // Output: Component Branch1: Perform operation

branch1.remove(leaf2);
branch2.operation(); // Output: Component Branch2: Perform operation

root.getChild(0).operation(); // Output: Component Branch1: Perform operation
```

In the code above, we have an abstract component Component, and we extended the functionality of the abstract component by creating two concrete components: Leaf and Composite. Composite holds an array of child objects and implemented the ability to include other components. We then used all these components to build a tree structure model. The parent node model is a Component object, whereas child nodes can be Component objects or Composite objects. Finally, we can perform operations by calling the operation method.

## lyweight Pattern

The Flyweight Pattern is a structural design pattern that minimizes memory usage and the number of class instances by sharing objects. Here’s a code sample of the Flyweight Pattern:

```
// Flyweight factory class
class FlyweightFactory {
  constructor() {
    this.flyweights = {};
  }

  getFlyweight(key) {
    if (!this.flyweights[key]) {
      this.flyweights[key] = new ConcreteFlyweight(key);
    }

    return this.flyweights[key];
  }
}

// Concrete Flyweight class
class ConcreteFlyweight {
  constructor(key) {
    this.key = key;
  }

  operation() {
    console.log(`ConcreteFlyweight ${this.key}: Perform operation`);
  }
}

// Utilizing the Flyweight Pattern
const factory = new FlyweightFactory();
const flyweight1 = factory.getFlyweight('key');
const flyweight2 = factory.getFlyweight('key');
flyweight1.operation(); // Output: ConcreteFlyweight key: Perform operation
flyweight2.operation(); // Output: ConcreteFlyweight key: Perform operation
console.log(flyweight1 === flyweight2); // Output: true
```

In the code above, we have a Flyweight factory class FlyweightFactory, used to create and manage the shared ConcreteFlyweight objects. Objects of ConcreteFlyweight contain data or states that need to be shared. We instantiate the FlyweightFactory, get objects through the getFlyweight() method of the FlyweightFactory, and verify whether the same object is shared among multiple objects. Ultimately, it’s demonstrated that flyweight1 and flyweight2 point to the same object, thereby upholding the concept of shared objects.

## Strategy Pattern

The Strategy Pattern is a design pattern that defines a series of algorithms and encapsulates each one, making them interchangeable. The Strategy Pattern allows the algorithm to change independently of the client that uses it. This pattern is a behavioral pattern. Here’s the sample code:

```
class Strategy {
  constructor(name) {
    this.name = name;
  }
  
  execute() {}
}

class StrategyA extends Strategy {
  execute() {
    console.log('Executing strategy A');
  }
} 

class StrategyB extends Strategy {
  execute() {
    console.log('Executing strategy B');
  }
}

class Context {
  constructor(strategy) {
    this.strategy = strategy;
  }
  
  executeStrategy() {
    this.strategy.execute();
  }
}

let context = new Context(new StrategyA('A'));
context.executeStrategy(); // Executing strategy A
 
context.strategy = new StrategyB('B');
context.executeStrategy(); // Executing strategy B
```

## Template Method Pattern

The Template Method Pattern is a behavioral design pattern. It defines the skeleton of an algorithm in an operation and delays some steps to subclasses. The Template Method Pattern allows a subclass to redefine certain steps of an algorithm without changing the algorithm’s structure.

Sample code:

```
class Game {
  setup() {}
  
  start() {
    this.setup();
    this.play();
    this.finish();
  }
  
  play() {}
  
  finish() {}
}

class Chess extends Game {
  setup() {
    console.log('Setting up chess game');
  }
  
  play() {
    console.log('Playing chess');
  }
  
  finish() {
    console.log('Finishing chess game');
  }
}

class TicTacToe extends Game {
  setup() {
    console.log('Setting up TicTacToe game');
  }
  
  play() {
    console.log('Playing TicTacToe');
  }
  
  finish() {
    console.log('Finishing TicTacToe game');
  }
}

let game = new Chess();
game.start();

game = new TicTacToe();
game.start();
```

## Observer Pattern

The Observer Pattern is a behavioral design pattern in which a one-to-many dependency exists between objects. When the state of an object changes, all its dependents get notified and automatically updated. The Observer pattern decouples relationships between objects, allowing them to change independently.

Sample code:

```
class Subject {
  constructor() {
    this.observers = [];
  }
  
  attach(observer) {
    this.observers.push(observer);
  }
  
  detach(observer) {
    const index = this.observers.indexOf(observer);
    if (index > -1) {
      this.observers.splice(index, 1);
    }
  }
  
  notify() {
    for(const observer of this.observers) {
      observer.update(this);
    }
  }
}

class Observer {
  update(subject) {}
}

class ConcreteSubject extends Subject {
  constructor(state) {
    super();
    this.state = state;
  }
  
  set_state(state) {
    this.state = state;
    this.notify();
  }
  
  get_state() {
    return this.state;
  }
} 

class ConcreteObserver extends Observer {
  update(subject) {
    console.log(`Got updated value: ${subject.get_state()}`);
  }
}

let subject = new ConcreteSubject('initial state');
let observer = new ConcreteObserver();

subject.attach(observer);
subject.set_state('new state');
```

## Iterator Pattern

The Iterator Pattern is a behavioral design pattern that provides a way to access the elements of a collection object in sequence. The Iterator Pattern delegates the responsibility of traversing the collection to the iterator instead of the collection itself. This allows for the implementation of the collection and the traversal algorithm to be decoupled, providing better flexibility.

Sample code:

```
class Iterator {
  constructor(items) {
    this.items = items;
    this.cursor = 0;
  }
  
  has_next() {
    return this.cursor < this.items.length;
  }
  
  next() {
    const item = this.items[this.cursor];
    this.cursor += 1;
    return item;
  }
}

class Collection {
  constructor() {
    this.items = [];
  }
  
  add_item(item) {
    this.items.push(item);
  }
  
  iterator() {
    return new Iterator(this.items);
  }
}

const collection = new Collection();
collection.add_item('item 1');
collection.add_item('item 2');
collection.add_item('item 3');

const iterator = collection.iterator();
while(iterator.has_next()) {
  console.log(iterator.next());
}
```

## Chain of Responsibility Pattern

The Chain of Responsibility Pattern is a behavioral design pattern. It allows multiple objects to have a chance to handle requests, thus avoiding coupling the sender and receiver of the request. These objects are linked into a chain, and the request is passed along this chain until one object handles it.

Implementation:

```
class Handler {
  constructor() {
    this.nextHandler = null;
  }

  setNextHandler(handler) {
    this.nextHandler = handler;
  }

  handleRequest(request) {
    if (this.nextHandler !== null) {
      return this.nextHandler.handleRequest(request);
    }
    return null;
  }
}

class ConcreteHandlerA extends Handler {
  handleRequest(request) {
    if (request === 'A') {
      return `Handle Request ${request}`;
    }
    return super.handleRequest(request);
  }
}

class ConcreteHandlerB extends Handler {
  handleRequest(request) {
    if (request === 'B') {
      return `Handle Request ${request}`;
    }
    return super.handleRequest(request);
  }
}

class ConcreteHandlerC extends Handler {
  handleRequest(request) {
    if (request === 'C') {
      return `Handle Request ${request}`;
    }
    return super.handleRequest(request);
  }
}

const handlerA = new ConcreteHandlerA();
const handlerB = new ConcreteHandlerB();
const handlerC = new ConcreteHandlerC();

handlerA.setNextHandler(handlerB);
handlerB.setNextHandler(handlerC);

console.log(handlerA.handleRequest('A')); // Handle Request A
console.log(handlerA.handleRequest('B')); // Handle Request B
console.log(handlerA.handleRequest('C')); // Handle Request C
console.log(handlerA.handleRequest('D')); // null
```

## Command Pattern

The Command Pattern is a behavioral design pattern that encapsulates a request or operation into an object, thus allowing you to decouple the initiator of the request or operation from the specific executor. Command Pattern can parameterize requests or operations, even dynamically combining commands at runtime.

Implementation:

```
class Command {
  constructor(receiver) {
    this.receiver = receiver;
  }

  execute() {
    throw new Error('You have to implement the method execute!');
  }
}

class ConcreteCommandA extends Command {
  execute() {
    this.receiver.actionA();
  }
}

class ConcreteCommandB extends Command {
  execute() {
    this.receiver.actionB();
  }
}

class Receiver {
  actionA() {
    console.log('Receiver Action A.');
  }

  actionB() {
    console.log('Receiver Action B.');
  }
}

class Invoker {
  constructor() {
    this.commands = new Map();
  }

  setCommand(key, command) {
    this.commands.set(key, command);
  }

  executeCommand(key) {
    const command = this.commands.get(key);
    if (!command) {
      console.log(`Command ${key} is not found.`);
      return;
    }
    command.execute();
  }
}

const receiver = new Receiver();
const invoker = new Invoker();

invoker.setCommand('A', new ConcreteCommandA(receiver));
invoker.setCommand('B', new ConcreteCommandB(receiver));

invoker.executeCommand('A'); // Receiver Action A.
invoker.executeCommand('B'); // Receiver Action B.
```

## Memento Pattern

The Memento Pattern is a behavioral design pattern that allows you to save and restore an object’s state without exposing the details of the object’s implementation. The Memento Pattern involves three roles: Memento, Originator, and Caretaker.

Implementation:

```
class Memento {
  constructor(state) {
    this.state = state;
  }

  getState() {
    return this.state;
  }
}

class Originator {
  constructor(state) {
    this.state = state;
  }

  setState(state) {
    this.state = state;
  }

  createMemento() {
    return new Memento(this.state);
  }

  restoreMemento(memento) {
    this.state = memento.getState();
  }

  getState() {
    return this.state;
  }
}

class Caretaker {
  constructor() {
    this.mementos = [];
  }

  addMemento(memento) {
    this.mementos.push(memento);
  }

  getMemento(index) {
    return this.mementos[index];
  }
}

const originator = new Originator('State A');
const caretaker = new Caretaker();

// Save state
caretaker.addMemento(originator.createMemento());

// change state
originator.setState('State B');

console.log(`Current State: ${originator.getState()}`);

// Restore state
originator.restoreMemento(caretaker.getMemento(0));

console.log(`Current State: ${originator.getState()}`);
```

## State Pattern

The State Pattern is a behavioral design pattern that allows an object to change its behavior when its internal state changes. The State Pattern encapsulates each state into a class, making any operations for that state can be handled within that class. This allows the state transition code to be abstracted from the main business logic, avoiding a large number of if-else statements.

Implementation:

```
class Context {
  constructor() {
    this.state = new ConcreteStateA(this);
  }

  setState(state) {
    this.state = state;
  }

  request() {
    this.state.handle();
  }
}

class State {
  constructor(context) {
    this.context = context;
  }

  handle() {
    throw new Error('You have to implement the method handle!');
  }
}

class ConcreteStateA extends State {
  handle() {
    console.log('Handle State A');
    this.context.setState(new ConcreteStateB(this.context));
  }
}

class ConcreteStateB extends State {
  handle() {
    console.log('Handle State B');
    this.context.setState(new ConcreteStateA(this.context));
  }
}

const context = new Context();
context.request(); // Handle State A
context.request(); // Handle State B
context.request(); // Handle State A
```

## Visitor Pattern

The Visitor Pattern is a behavioral design pattern that allows you to encapsulate an algorithm in one or more visitor classes, allowing you to define new operations that act on these elements without changing the interfaces of each element class.

Implementation:

```
class Element {
  accept(visitor) {
    throw new Error('You have to implement the method accept!');
  }
}

class ConcreteElementA extends Element {
  accept(visitor) {
    visitor.visitConcreteElementA(this);
  }

  operationA() {
    console.log('Operation A of Concrete Element A.');
  }
}

class ConcreteElementB extends Element {
  accept(visitor) {
    visitor.visitConcreteElementB(this);
  }

  operationB() {
    console.log('Operation B of Concrete Element B.');
  }
}

class Visitor {
  visitConcreteElementA(element) {
    console.log(`Visit Concrete Element A with ${element.operationA()}`);
  }

  visitConcreteElementB(element) {
    console.log(`Visit Concrete Element B with ${element.operationB()}`);
  }
}

const elementA = new ConcreteElementA();
const elementB = new ConcreteElementB();
const visitor = new Visitor();

elementA.accept(visitor);
elementB.accept(visitor);
```

## Mediator Pattern

The Mediator Pattern is a behavioral design pattern that allows you to reduce direct dependencies between components by having them interact through a mediator. By avoiding explicit referencing each other between components, the mediator would allow components to be more easily reused.

Implementation:

```
class Mediator {
  constructor() {
    this.components = new Set();
  }

  register(component) {
    component.mediator = this;
    this.components.add(component);
  }

  notify(sender, event) {
    this.components.forEach((component) => {
      if (component !== sender) {
        component.receive(sender, event);
      }
    });
  }
}

class Component {
  constructor(name) {
    this.name = name;
    this.mediator = null;
  }

  send(event) {
    console.log(`Send event ${event} from ${this.name}`);
    this.mediator.notify(this, event);
  }

  receive(sender, event) {
    console.log(`Receive event ${event} from ${sender.name} by ${this.name}`);
  }
}

const mediator = new Mediator();
const componentA = new Component('Component A');
const componentB = new Component('Component B');
const componentC = new Component('Component C');
mediator.register(componentA);
mediator.register(componentB);
mediator.register(componentC);

componentA.send('Hello');  //Send event Hello from Component A, Receive event Hello from Component A by Component B, Receive event Hello from Component A by Component C
```

## nterpreter Pattern

The Interpreter Pattern is a behavioral design pattern that can represent the grammar of a language (typically a programming language) or expression as a parse tree, and define an interpreter to interpret this language or expression.

Implementation:

```
class Context {
  constructor(input) {
    this.input = input;
    this.output = 0;
  }
}

class Expression {
  interpreter(context) {
    throw new Error('You have to implement the method interpreter!');
  }
}

class ThousandExpression extends Expression {
  interpreter(context) {
    const str = context.input;
    if (str.startsWith('M')) {
      context.output += 1000;
      context.input = str.slice(1);
    }
  }
}

class HundredExpression extends Expression {
  interpreter(context) {
    const str = context.input;
    if (str.startsWith('C')) {
      context.output += 100;
      context.input = str.slice(1);
    } else if (str.startsWith('CD')) {
      context.output += 400;
      context.input = str.slice(2);
    } else if (str.startsWith('CM')) {
      context.output += 900;
      context.input = str.slice(2);
    }
  }
}

class TenExpression extends Expression {
  interpreter(context) {
    const str = context.input;
    if (str.startsWith('X')) {
      context.output += 10;
      context.input = str.slice(1);
    } else if (str.startsWith('XL')) {
      context.output += 40;
      context.input = str.slice(2);
    } else if (str.startsWith('XC')) {
      context.output += 90;
      context.input = str.slice(2);
    }
  }
}

class OneExpression extends Expression {
  interpreter(context) {
    const str = context.input;
    if (str.startsWith('I')) {
      context.output += 1;
      context.input = str.slice(1);
    } else if (str.startsWith('IV')) {
      context.output += 4;
      context.input = str.slice(2);
    } else if (str.startsWith('V')) {
      context.output += 5;
      context.input = str.slice(1);
    } else if (str.startsWith('IX')) {
      context.output += 9;
      context.input = str.slice(2);
    }
  }
}

class Interpreter {
  static parse(roman) {
    const context = new Context(roman);
    const tree = [
      new ThousandExpression(),
      new HundredExpression(),
      new TenExpression(),
      new OneExpression(),
    ];
    tree.forEach((expression) => expression.interpreter(context));
    return context.output;
  }
}

console.log(Interpreter.parse('CDXLVIII')); // 448
```
