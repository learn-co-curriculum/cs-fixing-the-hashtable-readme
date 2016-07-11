# cs-fixing-the-hashtable-readme

## Learning goals

1.  Fix a performance bug.
2.  Use UML class diagrams to represent relationships between classes.


## Overview

This README presents our solution to the previous lab: a map implementation called `MyFixedHashMap` because it fixes the performance bug in `MyHashMap`.  We'll also introduce the UML class diagram, which is a graphical representation of the relationships between classes.


## Fixing `MyHashMap`

The problem with `MyHashMap` is in `size`, which is inherited from `MyBetterMap`:

```java
	public int size() {
		int total = 0;
		for (MyLinearMap<K, V> map: maps) {
			total += map.size();
		}
		return total;
	}
```

To add up the total size it has to iterate the sub-maps.  Since we increase the number of sub-maps, `k`, as the number of entries, `n`, increases, `k` is proportional to `n`, so `size` is linear.

And that makes `put` linear, too, because it uses `size`:

```java
	public V put(K key, V value) {
		V oldValue = super.put(key, value);

		if (size() > maps.size() * FACTOR) {
			rehash();
		}
		return oldValue;
	}
```

Everything we did to make `put` constant time is wasted if `size` is linear!

Fortunately, there is a simple solution, and we have seen it before: we have to keep the number of entries in an instance variable and update it whenever we call a method that changes it.

If you check out the `solution` branch of the previous lab, `javacs-lab08`, you'll find our solution in `MyFixedHashMap.java`.

Here's the beginning of the class definition:

```java
public class MyFixedHashMap<K, V> extends MyHashMap<K, V> implements Map<K, V> {

	private int size = 0;

	public void clear() {
		super.clear();
		size = 0;
	}
```

Rather than modify `MyHashedMap`, we define a new class that extends it.  It adds a new instance variable, `size`, which is initialized to zero.

Updating `clear` is straightforward; we invoke `clear` in the superclass (which clears the sub-maps), and then update `size`.

Updating `remove` and `put` is a little more difficult because when we invoke the method on the superclass, we can't tell whether the size of the sub-map changed.  Here's how we worked around that:


```java
	public V remove(Object key) {
		MyLinearMap<K, V> map = chooseMap(key);
		size -= map.size();
		V oldValue = map.remove(key);
		size += map.size();
		return oldValue;
	}
```

`remove` uses `chooseMap` to find the right sub-map, then subtracts away the size of the sub-map.  It invokes `remove` on the sub-map, which may or may not change the size of the sub-map, depending on whether it finds the key.  But either way, we add the new size of the sub-map back to `size`, so the final value of `size` is correct.

The rewritten version of `put` is similar:


```java
	public V put(K key, V value) {
		MyLinearMap<K, V> map = chooseMap(key);
		size -= map.size();
		V oldValue = map.put(key, value);
		size += map.size();

		if (size() > maps.size() * FACTOR) {
			size = 0;
			rehash();
		}
		return oldValue;
	}
```

We have the same problem here: when we invoke `put` on the sub-map, we don't know whether it added a new entry.  So we use the same solution, subtracting off the old size and then adding in the new size.

Now the implementation of the `size` method is simple:


```java
	public int size() {
		return size;
	}
```

And that's pretty clearly constant time.

When we profiled this solution, we found that the total time for putting `n` keys is proportional to `n`, which means that each `put` is constant time, as it's supposed to be.


## UML class diagrams

One challenge of working with the code for this lab is that we have several classes that depend on each other.  Here are some of the relationships between the classes:

*  `MyLinearMap` contains an `ArrayList` and implements `Map`.
*  `MyBetterMap` contains many `MyLinearMap` objects and implements `Map`.
*  `MyHashMap` extends `MyBetterMap`, so it also contains `MyLinearMap` objects, and it implements `Map`.
*  `MyFixedHashMap` also extends `MyHashMap`, and it implements `Map`.

To help keep track of relationships like these, software engineers often use "UML class diagrams".  UML stands for [Unified Modeling Language](https://en.wikipedia.org/wiki/Unified_Modeling_Language); a class diagram is one of several graphical standards defined by UML.

In a class diagram, each class is represented by a box, and relationships between classes are represented by arrows.  Here is a UML class diagram for the classes from the previous lab:

![alt tag](http://yuml.me/9e9efaa9)

Different relationships are represented by different arrows:

*  Arrows with a solid head indicate HAS-A relationships.  For example, each instance of `MyBetterMap` contains multiple instances of `MyLinearMap`, so they are connected by a solid arrow.
*  Arrows with a hollow head and a solid line indicate IS-A relationships.  For example, `MyHashMap` extends `MyBetterMap`, so they are connected by an IS-A arrow.
*  Arrows with a hollow head and a dashed line indicate that one class implements another; in this diagram, every class implements `Map` (except `Map` itself).

UML class diagrams provide a concise way to represent a lot of information about a collection of classes.  They are used during design phases to communicate about alternative designs, during implementation phases to maintain a shared mental map of the project, and during deployment to document the design.

## Resources

[yUML](http://yuml.me/) is the online tool we used to draw the diagram in this README.

<p class='util--hide'>View <a href='https://learn.co/lessons/cs-fixing-the-hashtable-readme'>Fixing the Hashtable</a> on Learn.co and start learning to code for free.</p>
