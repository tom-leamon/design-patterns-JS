# Design Patterns JS

**[Behavioral](#Behavioral)**
* [Chain Of Resp](#chain-of-resp) delegates commands to a chain of processing objects.
* [Command](#command)
* [Interpreter](#interpreter)
* [Iterator](#iterator)
* [Mediator](#mediator)
* [Memento](#memento)
* [Observer](#observer)
* [State](#state)
* [Strategy](#strategy)
* [Template](#template)
* [Visitor](#visitor)

**[Creational](#Creational)**
* [Abstract Factory](#abstract-factory)
* [Builder](#builder)
* [Factory](#factory)
* [Prototype](#prototype)
* [Singleton](#singleton)

**[Structural](#Structural)**
* [Adapter](#adapter)
* [Bridge](#bridge)
* [Composite](#composite)
* [Decorator](#decorator)
* [Facade](#facade)
* [Flyweight](#flyweight)
* [Proxy](#proxy)



## Behavioral
Most of these design patterns are specifically concerned with communication between objects.
### Chain Of Resp
Delegates commands to a chain of processing objects.
##### chain-of-resp-es6.js
```Javascript
class ShoppingCart {

  constructor() {
    this.products = [];
  }

  addProduct(p) {
    this.products.push(p);
  };
}

class Discount {

  calc(products) {
    let ndiscount = new NumberDiscount();
    let pdiscount = new PriceDiscount();
    let none = new NoneDiscount();
    ndiscount.setNext(pdiscount);
    pdiscount.setNext(none);
    return ndiscount.exec(products);
  };
}

class NumberDiscount {

  constructor() {
    this.next = null;
  }

  setNext(fn) {
    this.next = fn;
  };

  exec(products) {
    let result = 0;
    if (products.length > 3)
      result = 0.05;

    return result + this.next.exec(products);
  };
}

class PriceDiscount {

  constructor() {
    this.next = null;
  }

  setNext(fn) {
    this.next = fn;
  };

  exec(products) {
    let result = 0;
    let total = products.reduce((a, b) => a + b);

    if (total >= 500)
      result = 0.1;

    return result + this.next.exec(products);
  };
}

class NoneDiscount {
  exec() {
    return 0;
  };
}

export {
  ShoppingCart,
  Discount
};

```

### Command
Creates objects which encapsulate actions and parameters.
##### command_es6.js
```Javascript
class Cockpit {

  constructor(command) {
    this.command = command;
  }

  execute() {
    this.command.execute();
  }
}

class Turbine {

  constructor() {
    this.state = false;
  }

  on() {
    this.state = true;
  }

  off() {
    this.state = false;
  }
}

class OnCommand {

  constructor(turbine) {
    this.turbine = turbine;
  }

  execute() {
    this.turbine.on();
  }
}

class OffCommand {

  constructor(turbine) {
    this.turbine = turbine;
  }

  execute() {
    this.turbine.off();
  }
}

export {
  Cockpit,
  Turbine,
  OnCommand,
  OffCommand
};

```

### Interpreter
Implements a specialized language.
##### interpreter_es6.js
```Javascript
class Sum {

  constructor(left, right) {
    this.left = left;
    this.right = right;
  }

  interpret() {
    return this.left.interpret() + this.right.interpret();
  }
}

class Min {

  constructor(left, right) {
    this.left = left;
    this.right = right;
  }

  interpret() {
    return this.left.interpret() - this.right.interpret();
  }
}

class Num {

  constructor(val) {
    this.val = val;
  }

  interpret() {
    return this.val;
  }
}

export {
  Num,
  Min,
  Sum
};

```

### Iterator
Accesses the elements of an object sequentially without exposing its underlying representation.
##### iterator_es6.js
```Javascript
class Iterator {

  constructor(el) {
    this.index = 0;
    this.elements = el;
  }

  next() {
    return this.elements[this.index++];
  }

  hasNext() {
    return this.index < this.elements.length;
  }
}

export default Iterator;

```

### Mediator
Allows loose coupling between classes by being the only class that has detailed knowledge of their methods.
##### mediator_es6.js
```Javascript
class TrafficTower {

  constructor() {
    this.airplanes = [];
  }

  requestPositions() {
    return this.airplanes.map(airplane => {
      return airplane.position;
    });
  }
}

class Airplane {

  constructor(position, trafficTower) {
    this.position = position;
    this.trafficTower = trafficTower;
    this.trafficTower.airplanes.push(this);
  }

  requestPositions() {
    return this.trafficTower.requestPositions();
  }
}

export {
  TrafficTower,
  Airplane
};

```

### Memento
Provides the ability to restore an object to its previous state (undo).
##### memento_es6.js
```Javascript
class Memento {
  constructor(value) {
    this.value = value;
  }
}

const originator = {
  store: function(val) {
    return new Memento(val);
  },
  restore: function(memento) {
    return memento.value;
  }
};

class Caretaker {

  constructor() {
    this.values = [];
  }

  addMemento(memento) {
    this.values.push(memento);
  }

  getMemento(index) {
    return this.values[index];
  }
}

export {
  originator,
  Caretaker
};

```

### Observer
Is a publish/subscribe pattern which allows a number of observer objects to see an event.
##### observer_es6.js
```Javascript
class Product {
  constructor() {
    this.price = 0;
    this.actions = [];
  }

  setBasePrice(val) {
    this.price = val;
    this.notifyAll();
  }

  register(observer) {
    this.actions.push(observer);
  }

  unregister(observer) {
    this.actions = this.actions.filter(el => !(el instanceof observer));
  }

  notifyAll() {
    return this.actions.forEach(el => el.update(this));
  }
}

class Fees {
  update(product) {
    product.price = product.price * 1.2;
  }
}

class Proft {
  update(product) {
    product.price = product.price * 2;
  }
}

export {
  Product,
  Fees,
  Proft
};

```

### State
Allows an object to alter its behavior when its internal state changes.
##### state_es6.js
```Javascript
class OrderStatus {
  constructor(name, nextStatus) {
    this.name = name;
    this.nextStatus = nextStatus;
  }

  next() {
    return new this.nextStatus();
  }
}

class WaitingForPayment extends OrderStatus {
  constructor() {
    super('waitingForPayment', Shipping);
  }
}

class Shipping extends OrderStatus {
  constructor() {
    super('shipping', Delivered);
  }
}

class Delivered extends OrderStatus {
  constructor() {
    super('delivered', Delivered);
  }
}

class Order {
  constructor() {
    this.state = new WaitingForPayment();
  }

  nextState() {
    this.state = this.state.next();
  };
}

export default Order;

```

### Strategy
Allows one of a family of algorithms to be selected on-the-fly at runtime.
##### strategy_es6.js
```Javascript
class ShoppingCart {

  constructor(discount) {
    this.discount = discount;
    this.amount = 0;
  }

  checkout() {
    return this.discount(this.amount);
  }

  setAmount(amount) {
    this.amount = amount;
  }
}

function guestStrategy(amount) {
  return amount;
}

function regularStrategy(amount) {
  return amount * 0.9;
}

function premiumStrategy(amount) {
  return amount * 0.8;
}

export {
  ShoppingCart,
  guestStrategy,
  regularStrategy,
  premiumStrategy
};

```

### Template
Method defines the skeleton of an algorithm as an abstract class, allowing its subclasses to provide concrete behavior.
##### template_es6.js
```Javascript
class Tax {

  calc(value) {
    if (value >= 1000)
      value = this.overThousand(value);

    return this.complementaryFee(value);
  }

  complementaryFee(value) {
    return value + 10;
  }

}

class Tax1 extends Tax {

  constructor() {
    super();
  }

  overThousand(value) {
    return value * 1.1;
  }
}

class Tax2 extends Tax {

  constructor() {
    super();
  }

  overThousand(value) {
    return value * 1.2;
  }
}

export {
  Tax1,
  Tax2
};

```

### Visitor
Separates an algorithm from an object structure by moving the hierarchy of methods into one object.
##### visitor_es6.js
```Javascript
function bonusVisitor(employee) {
  if (employee instanceof Manager)
    employee.bonus = employee.salary * 2;
  if (employee instanceof Developer)
    employee.bonus = employee.salary;
}

class Employee {

  constructor(salary) {
    this.bonus = 0;
    this.salary = salary;
  }

  accept(visitor) {
    visitor(this);
  }
}

class Manager extends Employee {
  constructor(salary) {
    super(salary);
  }
}

class Developer extends Employee {
  constructor(salary) {
    super(salary);
  }
}

export {
  Developer,
  Manager,
  bonusVisitor
};

```


## Creational
Creational patterns are ones that create objects for you, rather than having you instantiate objects directly. This gives your program more flexibility in deciding which objects need to be created for a given case.
### Abstract Factory
Provide an interface for creating families of related or dependent objects without specifying their concrete classes.
##### abstract-factory_es6.js
```Javascript
function droidProducer(kind) {
  if (kind === 'battle') return battleDroidFactory;
  return pilotDroidFactory;
}

function battleDroidFactory() {
  return new B1();
}

function pilotDroidFactory() {
  return new Rx24();
}

class B1 {
  info() {
    return "B1, Battle Droid";
  }
}

class Rx24 {
  info() {
    return "Rx24, Pilot Droid";
  }
}

export default droidProducer;

```

### Builder
Separate the construction of a complex object from its representation, allowing the same construction process to create various representations.
##### builder_es6.js
```Javascript
class Request {
  constructor() {
    this.url = '';
    this.method = '';
    this.payload = {};
  }
}

class RequestBuilder {
  constructor() {
    this.request = new Request();
  }

  forUrl(url) {
    this.request.url = url;
    return this;
  }

  useMethod(method) {
    this.request.method = method;
    return this;
  }

  payload(payload) {
    this.request.payload = payload;
    return this;
  }

  build() {
    return this.request;
  }

}

export default RequestBuilder;

```

### Factory
Define an interface for creating a single object, but let subclasses decide which class to instantiate. Factory Method lets a class defer instantiation to subclasses.
##### factory_es6.js
```Javascript
class BmwFactory {

  static create(type) {
    if (type === 'X5')
      return new Bmw(type, 108000, 300);
    if (type === 'X6')
      return new Bmw(type, 111000, 320);
  }
}

class Bmw {
  constructor(model, price, maxSpeed) {
    this.model = model;
    this.price = price;
    this.maxSpeed = maxSpeed;
  }
}

export default BmwFactory;

```

### Prototype
Specify the kinds of objects to create using a prototypical instance, and create new objects from the 'skeleton' of an existing object, thus boosting performance and keeping memory footprints to a minimum.
##### prototype_es6.js
```Javascript
class Sheep {

  constructor(name, weight) {
    this.name = name;
    this.weight = weight;
  }

  clone() {
    return new Sheep(this.name, this.weight);
  }
}

export default Sheep;

```

### Singleton
Ensure a class has only one instance, and provide a global point of access to it.
##### singleton_es6.js
```Javascript
class Person {
  constructor() {
    if (typeof Person.instance === 'object') {
      return Person.instance;
    }
    Person.instance = this;
    return this;
  }
}

export default Person;

```


## Structural
These concern class and object composition. They use inheritance to compose interfaces and define ways to compose objects to obtain new functionality.
### Adapter
Allows classes with incompatible interfaces to work together by wrapping its own interface around that of an already existing class.
##### adapter_es6.js
```Javascript
class Soldier {
  constructor(level) {
    this.level = level;
  }

  attack() {
    return this.level * 1;
  }
}

class Jedi {
  constructor(level) {
    this.level = level;
  }

  attackWithSaber() {
    return this.level * 100;
  }
}

class JediAdapter {
  constructor(jedi) {
    this.jedi = jedi;
  }

  attack() {
    return this.jedi.attackWithSaber();
  }
}

export {
  Soldier,
  Jedi,
  JediAdapter
};

```

### Bridge
Decouples an abstraction from its implementation so that the two can vary independently.
##### bridge_es6.js
```Javascript
class Printer {
  constructor(ink) {
    this.ink = ink;
  }
}

class EpsonPrinter extends Printer {
  constructor(ink) {
    super(ink);
  }

  print() {
    return "Printer: Epson, Ink: " + this.ink.get();
  }
}

class HPprinter extends Printer {
  constructor(ink) {
    super(ink);
  }

  print() {
    return "Printer: HP, Ink: " + this.ink.get();
  }
}

class Ink {
  constructor(type) {
    this.type = type;
  }
  get() {
    return this.type;
  }
}

class AcrylicInk extends Ink {
  constructor() {
    super("acrylic-based");
  }
}

class AlcoholInk extends Ink {
  constructor() {
    super("alcohol-based");
  }
}

export {
  EpsonPrinter,
  HPprinter,
  AcrylicInk,
  AlcoholInk
};

```

### Composite
Composes zero-or-more similar objects so that they can be manipulated as one object.
##### composite_es6.js
```Javascript
//Equipment
class Equipment {

  getPrice() {
    return this.price || 0;
  }

  getName() {
    return this.name;
  }

  setName(name) {
    this.name = name;
  }
}

// --- composite ---
class Composite extends Equipment {

  constructor() {
    super();
    this.equipments = [];
  }

  add(equipment) {
    this.equipments.push(equipment);
  }

  getPrice() {
    return this.equipments.map(equipment => {
      return equipment.getPrice();
    }).reduce((a, b) => {
      return a + b;
    });
  }
}

class Cabinet extends Composite {
  constructor() {
    super();
    this.setName('cabinet');
  }
}

// --- leafs ---
class FloppyDisk extends Equipment {
  constructor() {
    super();
    this.setName('Floppy Disk');
    this.price = 70;
  }
}

class HardDrive extends Equipment {
  constructor() {
    super();
    this.setName('Hard Drive');
    this.price = 250;
  }
}

class Memory extends Equipment {
  constructor() {
    super();
    this.setName('Memory');
    this.price = 280;
  }
}

export {
  Cabinet,
  FloppyDisk,
  HardDrive,
  Memory
};

```

### Decorator
Dynamically adds/overrides behaviour in an existing method of an object.
##### decorator_es6.js
```Javascript
class Pasta {
  constructor() {
    this.price = 0;
  }
  getPrice() {
    return this.price;
  }
}

class Penne extends Pasta {
  constructor() {
    super();
    this.price = 8;
  }
}

class PastaDecorator extends Pasta {
  constructor(pasta) {
    super();
    this.pasta = pasta;
  }

  getPrice() {
    return this.pasta.getPrice();
  }
}

class SauceDecorator extends PastaDecorator {
  constructor(pasta) {
    super(pasta);
  }

  getPrice() {
    return super.getPrice() + 5;
  }
}

class CheeseDecorator extends PastaDecorator {
  constructor(pasta) {
    super(pasta);
  }

  getPrice() {
    return super.getPrice() + 3;
  }
}

export {
  Penne,
  SauceDecorator,
  CheeseDecorator
};

```

### Facade
Provides a simplified interface to a large body of code.
##### facade_es6.js
```Javascript
class ShopFacade {
  constructor() {
    this.discount = new Discount();
    this.shipping = new Shipping();
    this.fees = new Fees();
  }

  calc(price) {
    price = this.discount.calc(price);
    price = this.fees.calc(price);
    price += this.shipping.calc();
    return price;
  }
}

class Discount {

  calc(value) {
    return value * 0.9;
  }
}

class Shipping {
  calc() {
    return 5;
  }
}

class Fees {

  calc(value) {
    return value * 1.05;
  }
}

export default ShopFacade;

```

### Flyweight
Reduces the cost of creating and manipulating a large number of similar objects.
##### flyweight_es6.js
```Javascript
class Color {
  constructor(name) {
    this.name = name
  }
}

class colorFactory {
  constructor(name) {
    this.colors = {};
  }
  create(name) {
    let color = this.colors[name];
    if (color) return color;
    this.colors[name] = new Color(name);
    return this.colors[name];
  }
};

export {
  colorFactory
};

```

### Proxy
Provides a placeholder for another object to control access, reduce cost, and reduce complexity.
##### proxy_es6.js
```Javascript
class Car {
  drive() {
    return "driving";
  };
}

class CarProxy {
  constructor(driver) {
    this.driver = driver;
  }
  drive() {
    return (this.driver.age < 18) ? "too young to drive" : new Car().drive();
  };
}

class Driver {
  constructor(age) {
    this.age = age;
  }
}

export {
  Car,
  CarProxy,
  Driver
};

```



