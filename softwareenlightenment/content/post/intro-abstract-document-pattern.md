+++
author = "Jacob Hell"
title = "Don't Want to Create a Ton of Getters/Setters? Say Hello to the Abstract Document Pattern."
date = "2020-03-23"
description = "Introducing the Abstract Document Pattern."
tags = [
    "design",
    "abstract document"
]

+++

In this post, I talk about the Abstract Document pattern, which allows one to pseudo-dynamically create properties for a model. While keeping the type safety that statically typed languages like Java provide.

<!--more-->

### Type Safety

Type safety is an important concept for programming languages. More dynamically typed languages like JavaScript will try to execute almost anything you try at it. Even if an operation between two values will not work on paper. For example:

```javascript
var foo(){
    return "a" + 1;
}

foo();
// will return "a1"
```

The value of type safety could make developers decide to use statically typed languages.

### Downside of Statically Typed Languages

One disadvantage of using Statically-typed languages is the loss of flexibility when trying to add new values to a model or class.

For example, let's say I have a class called `Car` with two properties: `Paint` and `Trim`.

```java
public class Car {
    private String paint;
    private String trim;
    
    public String getPaint() {
       return paint;
    }
    
    public void setPaint(String paint) {
        this.paint = paint;
    }
    
    public String getTrim() {
        return trim;
    }
    
    public void setTrim(String trim) {
        this.trim = trim;
    }
}	
```

Looks good right? This class would be fine if we just wanted to hold onto paint and trim. But what if we want to add the following fields:

- Window tint
- Engine type
- Wheel model
- Radiator type
- Has CD Player
- Has Touch Screen
- Plus potentially 1000+ more fields, depending on what the customer is wanting.

I don't even want to think how big this class is going to get if we tried to implement all these fields!

### Implementing the Abstract Document

To describe the Abstract Document, first we must start with an interface that I call `Document`.

```java
public interface Document {
	void put(String key, Object value);

	Object get(String key);
}
```

Pretty simple interface. Not really different from the default `Map` abstract class in Java.

We will then create a class called `Abstract Document` which implements `Document`.

```java
public abstract class AbstractDocument implements Document {
	
	protected final Map<String, Object> properties;
	
	protected AbstractDocument(Map<String, Object> properties) {
		this.properties = properties;
	}
	
	@Override
	public void put(String key, Object value) {
		properties.put(key, value);
	}

	@Override
	public Object get(String key) {
		return properties.get(key);
	}
}
```

`AbstractDocument` is the class that all concrete documents will extend from. Again, this class is pretty simple. Things get exciting when we starting making more classes that extend `Document`. 

Before I go further, let me describe what we are trying to accomplish in this scenario. Going back to the `Car` class I had before, let's implement a workflow that will allow developers to create a Car that has a **Model**, **Price**, and list of **Parts**. These parts can have their own model and price, and also a **Type**.

### Implementing Our Use Case

We first begin by adding interfaces that extend `Document`. These interfaces should describe a noun that are a property of another noun. Let's start with the model of the car. It is a noun which is a property of the car.

```java
public interface HasModel extends Document {
	String getModel();
}
```

Another property we want to add is **Price**.

```java
public interface HasPrice extends Document {
	Number getPrice();
}
```

We discussed adding **Parts** as well. Before adding the interface, remember that each `Part` will have it's own **Model** and **Price** as well. It will also have a **Type**, something that a **Car** will not have. Let's go ahead and create the `HasType` interface.

```java
public interface HasType extends Document {
	String getType();
}
```

`Part` is going to be our first concrete class. This is where the magic of the **Abstract Document** is revealed:

```java
public class Part extends AbstractDocument implements HasType, HasModel, HasPrice {
	protected Part(Map<String, Object> properties) {
		super(properties);
	}

	@Override
	public Number getPrice() {
		return (Number) properties.get(Property.PRICE.toString());
	}

	@Override
	public String getModel() {
		return (String) properties.get(Property.MODEL.toString());
	}

	@Override
	public String getType() {
		return (String) properties.get(Property.TYPE.toString());
	}
}
```

When creating the concrete classes, remember that we must extend `AbstractDocument`. Every `Document` that we add as well we must implement. You can implement as many Documents you want.

Lastly, let's create the `Car` concrete class.

```java
public class Car extends AbstractDocument implements HasModel, HasPrice, HasParts {
	protected Car(Map<String, Object> properties) {
		super(properties);
	}

	@Override
	public List<Part> getParts() {
		@SuppressWarnings("unchecked")
		List<Map<String, Object>> parts = (List<Map<String, Object>>) properties.get(Property.PARTS.toString());
		
		return parts.stream().map(Part::new).collect(Collectors.toList());
	}

	@Override
	public Number getPrice() {
		return (Number) properties.get(Property.PRICE.toString());
	}

	@Override
	public String getModel() {
		return (String) properties.get(Property.MODEL.toString());
	}
}
```

### Bringing it all Together

Now that we have got the **Abstract Document** implemented with concrete classes, let's get a simple client working.

```java
public class Main {
	public static void main(String[] args) {
		var engine = Map.of(
				Property.MODEL.toString(), "Model 1",
				Property.PRICE.toString(), 200.00,
				Property.TYPE.toString(), "Engine"
		);
		
		var muffler = Map.of(
				Property.TYPE.toString(), "Muffler",
				Property.PRICE.toString(), 100.00,
				Property.MODEL.toString(), "Model 2"
		);
		
		var carProperties = Map.of(
				Property.MODEL.toString(), "Fast car",
				Property.PRICE.toString(), 50000,
				Property.PARTS.toString(), List.of(engine, muffler)
		);
		
		var car = new Car(carProperties);
		
		System.out.println(car.getModel());
		System.out.println(car.getPrice());
		car.getParts().forEach(part -> System.out.println(part.getModel()));
	}
}
```

Output:

````
Fast car
50000
Model 1
Model 2
````

### Conclusion

Thanks for reading!

I have created a GitHub repo with the source for this post. Please [go here](https://github.com/jakehell/AbstractDocumentPatternExample) to view it.