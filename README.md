# SOLID notes

My notes on the SOLID principles of object-oriented design, based mostly on [this article from scotch.io](https://scotch.io/bar-talk/s-o-l-i-d-the-first-five-principles-of-object-oriented-design) and [this article from Medium](https://medium.com/@cramirez92/s-o-l-i-d-the-first-5-priciples-of-object-oriented-design-with-javascript-790f6ac9b9fa).
Features examples rewritten in JavaScript (ES5 and ES6) instead of PHP.

## Table of Contents
1. [Single-responsibility principle](#s)
2. [Open-closed principle](#o)
3. [Liskov substitution principle](#l)
4. [Interface segregation principle](#i)

   4.5. [Digression on multiple interfaces/inheritance in JavaScript](#multinterfaces)
5. [Dependency inversion principle](#d)

## <a name="s"></a> Single-responsibility principle
> A class should have one and only one reason to change.

- **One job per class**.

- But you can also write wrappers to accomplish multiple things: e.g. `getShows()`, `getMovies()` and `getMusic()` -> `getUserMedia()` as a wrapper.

#### With factory functions:
```javascript
var Circle = function(radius) {
	var obj = { radius: radius };
	return obj;
};

var Square = function(length) {
	var obj = { length: length };
	return obj;
};

var AreaCalculator = function(shapesArray) {
	var obj = { _shapes: shapesArray };
	return obj;
}

AreaCalculator.prototype.sum = function() {
	var reduceToArea = function(prev, curr) {
		if (curr instanceof Square) {
			return prev + curr.length * curr.length;
		} else if (curr instanceof Circle) {
			return prev + Math.PI * (curr.radius * curr.radius)
		} else {
			throw new Error('Not a supported shape');
		}
	};

	return this._shapes.reduce(reduceToArea, 0);
}

AreaCalculator.prototype.output = function() {
	console.log('Sum of the areas of the provided shapes: ', this.sum());
}
```
#### With ES6 classes:
```javascript
class Circle {
	constructor(radius) {
		this.radius = radius;
	}
}

class Square {
	constructor(length) {
		this.length = length;
	}
}

class AreaCalculator {
	constructor(shapesArray) {
		this._shapes = shapesArray;
	}

	sum() {
		var reduceToArea = function(prev, curr) {
			if (curr instanceof Square) {
				return prev + curr.length * curr.length;
			} else if (curr instanceof Circle) {
				return prev + Math.PI * (curr.radius * curr.radius)
			} else {
				throw new Error('Not a supported shape');
			}
		};

		return this._shapes.reduce(reduceToArea, 0);
	}

	output() {
		console.log('Sum of the areas of the provided shapes: ', this.sum());
	}
}
```
#### Example usage:
```javascript
var shapes = [new Circle(2), new Square(5), new Square(6)];
var areas = new AreaCalculator(shapes);
areas.output();
```

The inclusion of an `output` method on `AreaCalculator` is problematic, as `AreaCalculator` handles the logic for outputting the data as well as for calculating it.

If the user wanted to output the data as another format, we would have to go back and modify the original code. For ease of extensibility, this method should not exist on `AreaCalculator`.

When we remove the `output` method, `AreaCalculator`'s **single responsibility** is summing the areas of the provided shapes.

To adhere to the **single responsibility principle**, create a `SumCalculatorOutputter` class to handle logic for outputting summed areas.

#### With factory functions:
```javascript
var SumCalculatorOutputter = function(areas) {
	var obj = { _areas: areas };
	return obj;
}

SumCalculatorOutputter.prototype.JSON = function() {};
SumCalculatorOutputter.prototype.HAML = function() {};
SumCalculatorOutputter.prototype.HTML = function() {};
SumCalculatorOutputter.prototype.JADE = function() {};
```
#### With ES6 classes:
```javascript
class SumCalculatorOutputter {
	constructor(areas) {
		this._areas = areas;
	}

	// methods for handling outputting logic in various formats!
	JSON() {}

	HAML() {}

	HTML() {}

	JADE() {}
}
```
#### Example usage:
```javascript
var shapes = [ new Circle(2), new Square(5), new Square(6) ];
var areas = new AreaCalculator(shapes);
var output = new SumCalculatorOutputter(areas);

output.JSON();
output.HAML();
output.HTML();
output.JADE();
```

## <a name="o"></a> Open-closed principle
> Objects or entities should be **open for extension, but closed for modification.**

- You should be able to easily extend the class without having to modify the core logic of the class itself.

- In the `sum` example from earlier, calculating a shape's area is accomplished by a bunch of if/else checks. To support more shapes, we'd have to add more if/else blocks. However, that would go against the **open-closed principle**.

Preserve extensibility by moving the area-calculation logic from the `sum` method to the shape classes themselves:

#### With factory functions:
```javascript
var Circle = function(radius) {
	var obj = { radius: radius };
	return obj;
};

Circle.prototype.area = function() {
	return Math.PI * Math.pow(this.radius, 2);
};

var Square = function(length) {
	var obj = { length: length };
	return obj;
};

Square.prototype.area = function() {
	return Math.pow(this.length, 2);
};

var AreaCalculator = function(shapesArray) {
	var obj = { _shapes: shapesArray };
	return obj;
};

AreaCalculator.prototype.sum = function() {
	return this._shapes.reduce(function(prev, curr) {
		return prev + curr.area();
	}, 0);
};
```
#### With ES6 classes:
```javascript
class Square {
	constructor(length) {
		this.length = length;
	}

	area() {
		return Math.pow(this.length, 2);
	}
}

class Circle {
	constructor(radius) {
		this.radius = radius;
	}

	area() {
		return Math.PI * Math.pow(this.radius, 2);
	}
}

class AreaCalculator {
	// ...

	sum() {
		return this._shapes.reduce(function(prev, curr) {
			return prev + curr.area();
		}, 0);
	}
}
```

How do we ensure that objects passed into our refactored `AreaCalculator` are shapes and have a method named `area`?

**Code to an interface that every shape extends.**

#### With factory functions:
```javascript
var shapeInterface = function(state) {
	return Object.assign({}, {
		type: 'shapeInterface',
		area: function() {
			return state.area(state);
		}
	});
};

var Square = function(length) {
	var proto = {
		length: length,
		type: 'Square',
		area: function(args) {
			return Math.pow(args.length, 2);
		}
	};
	var basics = shapeInterface(proto);
	var composite = Object.assign({}, basics);
	return Object.assign(Object.create(composite), { length: length });
}
```
#### With ES6 classes:
```javascript
class ShapeInterface {
	area() {}
}

class Circle extends ShapeInterface {
	constructor(radius) {
		this.radius = radius;
	}

	area() {
		return Math.PI * Math.pow(this.radius, 2);
	}
}
```
In the `sum` method of `AreaCalculator`, we can check if the shapes provided are actually instances of `ShapeInterface`, and throw an exception if that is not the case.

#### With factory functions:
```javascript
AreaCalculator.prototype.sum = function() {
	return this._shapes.reduce(function(prev, curr) {
		if (!Object.getPrototypeOf(curr) instanceof ShapeInterface) {
			throw new Error('AreaCalculatorInvalidShapeException');

		}
		return prev + curr.area();
	}, 0);
};
```

#### With ES6 classes:
```javascript
class AreaCalculator {
	// ...
	sum() {
		return this._shapes.reduce(function(prev, curr) {
			if (!Object.getPrototypeOf(curr) instanceof ShapeInterface) {
				throw new Error('AreaCalculatorInvalidShapeException');
			}
			return prev + curr.area();
		}, 0);
	}
}
```
## <a name="l"></a> Liskov substitution principle
> Let q(x) be a property provable about objects of x of type T. Then q(y) should be provable for objects y of type S, where S is a subtype of T.

- Every subclass should be substitutable for its base/parent class.

- When overriding parent methods in a subclass, do it in a way that *does not break functionality* from a client's POV.

In the example below, the `VolumeCalculator`class extends the `AreaCalculator` class and thus should be substitutable for it.

#### With factory functions:
```javascript
const areaCalculator = (s) => {
	const proto = {
		sum() {
			const area = [];
			for (shape of this.shapes) {
				area.push(shape.area());
			}
			return area.reduce((v, c) => c += v, 0);
		},
		output() {}
	};

	return Object.assign(Object.create(proto), { shapes: s });
};

const volumeCalculator = (s) => {
	const proto = {
		type: 'volumeCalculator'
	};
	const areaCalProto = Object.getPrototypeOf(areaCalculator());
	const inherit = Object.assign({}, areaCalProto, proto);

	return Object.assign(Object.create(inherit), { shapes: s });
};
```
#### With ES6 classes:
```javascript
class VolumeCalculator extends AreaCalculator {
	constructor(shapesArray) {
		this._shapes = shapesArray;
	}

	sum() {
		// logic to calculate volumes + return a float, double, or integer, so that this sum method is like the sum method from the AreaCalculator.
	}
}

class SumCalculatorOutputter {
	constructor(calculator){
		this.calculator = calculator;
	}

	JSON() {
		var data = this.calculator.sum();
		return JSON.toJSON(data);
	}

	HTML() {
		//...
	}
}
```
#### Example usage:
```javascript
var areas = new AreaCalculator(shapes);
var volumes = new AreaCalculator(solidShapes);

var output = new SumCalculatorOutputter(areas);
var output2 = new SumCalculatorOutputter(volumes);
```
## <a name="i"></a> Interface segregation principle

> A client should never be forced to implement or depend on an interface/method that it doesn't use.

- JS doesn't have interfaces, so use **function composition + closures to simulate interfaces**.

If we add the `volume` method to  `shapeInterface`, then the `Square` class will implement a method it has no use of - the `volume` method.

To avoid this, create a separate interface for this - *solid shapes* will implement the new interface.

#### With ES6 factory functions:
```javascript
const shapeInterface = (state) => ({
	type: 'shapeInterface',
	area: () => state.area(state)
});

const square = (length) => {
	const proto = {
		length,
		type: 'Square',
		area: (args) => Math.pow(args.length, 2)
	};

	// call the shapeInterface factory function in the factory function for Square

	const basics = shapeInterface(proto);
	const composite = Object.assign({}, basics);
	return Object.assign(Object.create(composite), { length }); // Object.create takes as first arg a prototype object
}
```
#### With ES5 factory functions:
```javascript
var shapeInterface = function(state) {
	return Object.assign({}, {
		type: 'shapeInterface',
		area: state.area(state)
	});
}

var Square = function(length) {
	var proto = {
		length: length,
		type: 'Square',
		area: function(args) {
			return Math.pow(args.length, 2);
		}
	};

	var basics = shapeInterface(proto);
	var composite = Object.assign({}, basics);
	return Object.assign(Object.create(composite), { length: length });
}
```
#### Example usage:
```javascript
const s = square(5);
console.log('OBJ', s); // => OBJ { length: 5 }
console.log('PROTO', Object.getPrototypeOf(s)); // => PROTO { type: 'shapeInterface', area: [Function: area] }
s.area() // => 25
```

Check in `areaCalculator`'s `sum` method whether the shapes provided inherit from the appropriate interface.

```javascript
var areaCalculator() {
	//...
	sum() {
		var area = [];
		for (var s in this.shapes) {
			if (Object.getPrototypeOf(shape).type === 'shapeInterface') {
				area.push(shape.area());
			} else {
				throw new Error('not a shapeInterface object');
			}
		}

		return area.reduce(function(v, c) { return c += v }, 0);
	}
}
```

### <a name="multinterfaces"></a> Digression on implementing multiple interfaces
There is no direct corollary to implementing multiple interfaces in JavaScript, but my mind jumps to multiple inheritance. To achieve multiple inheritance in JS, you have options including:

 1. Have a constructor fn [call more than one other constructor fn within it](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Details_of_the_Object_Model#No_multiple_inheritance). Unfortunately, the result is that the object will not be responsive to changes in the constructor's prototype.
 2. [Use mixins](https://javascriptweblog.wordpress.com/2011/05/31/a-fresh-look-at-javascript-mixins/):

   * Create an object literal mixin with all the properties you want to inherit. Then use an `extend` or `augment` function to copy the mixin's fns into the reeiving objects.

   * Alternatively, create a functional mixin and have the descendant objects call the functions on themselves (`mixinClass1.call(inheritingClass.prototype); mixinClass2.call(inheritingClass.prototype);`).

 3. [Use proxy objects in ES6](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

In the examples below, the approach is closest to 2: properties from both `shapeInterface` and `solidShapeInterface`, as well as the ultimate `type` of `solidShapeInterface`, are copied into the `composite` that `Cuboid` inherits from.

#### With factory functions:
```javascript
var shapeInterface = function(state) {
	return Object.assign({}, {
		type: 'shapeInterface',
		area: state.area(state)
	});
};

var solidShapeInterface = function(state) {
	return Object.assign({}, {
		type: 'solidShapeInterface',
		volume: state.volume(state)
	});
};

var Cuboid = function(length) {
	var proto = {
		length: length,
		type: 'Cuboid',
		area: function(args) {
			return Math.pow(args.length, 2) * 6;
		},
		volume: function(args) {
			return Math.pow(args.length, 3);
		}
	};
	var shapeProto = shapeInterface(proto);
	var solidShapeProto = solidShapeInterface(proto);
	var composite = Object.assign({}, shapeProto, solidShapeProto);

	return Object.assign(Object.create(composite), { length: length });
}
```
To expose a single API for managing both shapes, implement a separate interface, `ManageShapeInterface`, on both the flat and solid shapes.

#### With factory functions:
```javascript
var manageShapeInterface = function(state) {
	return Object.assign({}, {
		type: 'manageShapeInterface',
		calculate: state.calculate(state)
	});
}

var Square = function(length) {
	var proto = {
		length: length,
		type: 'Square',
		area: function(args) {
			return Math.pow(args.length, 2);
		},
		calculate: function(args) {
			return this.area(args);
		}
	};

	var shapeProto = shapeInterface(proto);
	var unifiedShapeInterfaceProto = manageShapeInterface(proto);
	var composite = Object.assign({}, shapeProto, unifiedShapeInterfaceProto);

	return Object.assign(Object.create(composite), {
		length: length
	});
}

var Cuboid = function(length) {
	var proto = {
		length: length,
		type: 'Cuboid',
		area: function(args) {
			return Math.pow(args.length, 2) * 6;
		},
		volume: function(args) {
			return Math.pow(args.length, 3);
		},
		calculate: function(args) {
			return this.area(args) + this.volume(args);
		}
	};

	var shapeProto = shapeInterface(proto);
	var solidShapeProto = solidShapeInterface(proto);
	var unifiedShapeInterfaceProto = manageShapeInterface(proto);
	var composite = Object.assign({}, shapeProto, solidShapeProto, unifiedShapeInterfaceProto);

	return Object.assign(Object.create(composite), { length: length });
}
```
Once we have a unified shape interface that abstracts over underlying differences, we reflect the abstracted API changes in `areaCalculator`:

```javascript
var areaCalculator() {
	//...
	sum() {
		var area = [];
		for (var s in this.shapes) {
			// check for manageShapeInterface as a proto
			if (Object.getPrototypeOf(shape).type === 'manageShapeInterface') {
				// replace call to area with calculate
				area.push(shape.calculate());
			} else {
				throw new Error('not a manageShapeInterface object');
			}
		}

		return area.reduce(function(v, c) { return c += v }, 0);
	}
}
```

Another approach, from the Medium article, takes advantage of functional composition: the `manageShapeInterface` factory function receives a higher-order function that decouples for every shape the functionality of getting the appropriate calculation.

```javascript
var manageShapeInterface = function(fn) {
	return Object.assign({}, {
		type: 'manageShapeInterface',
		calculate: function() { return fn(); }
	});
}

var Circle = function(radius) {
	var proto = {
		radius: radius,
		type: 'Circle',
		area: function(args) {
			return Math.PI * Math.pow(args.radius, 2)
		}
	};

	var basics = shapeInterface(proto);
	var abstraction = manageShapeInterface(function() { return basics.area() });
	var composite = Object.assign({}, basics, abstraction);

	return Object.assign(Object.create(composite), { radius: radius });
}

var Cuboid = function(length) {
	var proto = {
		length: length,
		type: 'Cuboid',
		area: function(args) { return Math.pow(args.length, 2) },
		volume: function(args) { return Math.pow(args.length, 3) }
	};

	var basics = shapeInterface(proto);
	var complex = solidShapeInterface(proto);
	var abstraction = manageShapeInterface(function() { return basics.area() + complex.volume(); });
	var composite = Object.assign({}, basics, abstraction);

	return Object.assign(Object.create(composite), { length: length });
}
```

## <a name="d"></a> Dependency inversion principle
> Entities must depend on abstractions, not concretions.

- Allows for decoupling. Note that JavaScript is a dynamic lang + doesn't require the use of abstractions to achieve decoupling.

- **High-level modules must depend on abstractions, and not on low-level modules.**

Below, `MySQLConnection` is a low-level module, and `PasswordReminder` is a high-level module. The `PasswordReminder` class depends on the `MySQLConnection` class.

The current setup is also a violation of the open-closed principle, as the code should be agnostic to database engines to remain easily modifiable.

#### With ES6 classes:
```javascript
class PasswordReminder {
	constructor(dbConnection) {
	this._dbConnection = dbConnection;
	}
}

class MySQLConnection {
	constructor(address) {
		this._address = address;
	}
}
```

Coding to an interface fixes this dependency and now both the high- and low-level modules depend on abstraction.

#### With factory functions:
```javascript
var dbConnectionInterface = function(state) {
	return Object.assign({}, {
		type: 'dbConnectionInterface',
		connect: state.connect(state)
	});
}

var MySQLConnection = function(address) {
	var proto = {
		address: address,
		type: 'MySQLConnection',
		connect: function() {
			return 'db connection';
		}
	};
	var basics = dbConnectionInterface(proto);
	var composite = Object.assign({}, basics);
	return Object.assign(Object.create(composite), { address: address });
}

var PasswordReminder = function(dbConnection) {
	return Object.assign({}, {
		dbConnection: dbConnection,
		type: 'PasswordReminder'
	});
}
```
#### With ES6 classes:
```javascript
class DBConnectionInterface {
	connect() {}
}

class MySQLConnection extends DBConnectionInterface {
	connect() {
		return 'db connection';
	}
}

class PasswordReminder {
	constructor(dbConnection) { // here, the dbConnection being passed in is any db connection that extends the DBConnectionInterface class. Therefore, the app can switch databases easily without needing extensive modifications.
		this.dbConnection = dbConnection;
	}
}
```
