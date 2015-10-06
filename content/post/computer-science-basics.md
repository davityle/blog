+++
Categories = ["Development", "Java", "CS"]
Description = "The fundamentals of Computer Science"
Tags = ["java", "dev", "algorithms", "data", "structures", "big-o"]
date = "2015-09-15T14:27:09-06:00"
menu = "main"
title = "Computer Science Basics by Example"
draft = true

+++


## Data Structures - Java

http://cs.brown.edu/cgc/jdsl/

### Hashmap
```
import java.util.Optional;

@SuppressWarnings("unchecked")
public class HashMap<T, R> {

    private static final int MIN_CAPACITY = 2;
    private Entry<T, R>[] table;
    private int internalSize, size;
    private float fillRatio;

    public HashMap() {
        this(MIN_CAPACITY);
    }

    public HashMap(int initialCapacity) {
        this(initialCapacity, .75f);
    }

    public HashMap(int initialCapacity, float fillRatio) {
        this.table = new Entry[Math.max(MIN_CAPACITY, initialCapacity)];
        this.fillRatio = fillRatio;
    }

    public Optional<R> put(T key, R value) {
        int index = getIndex(key);
        Entry<T, R> current = table[index];
        table[index] = new Entry<>(key, value);

        if(value == null && current != null && current.getValue() != null) {
            size--;
        } else if(value != null && (current == null || current.getValue() == null)){
            size++;
        }

        if(current == null && ++internalSize >= (table.length * fillRatio)) {
            resizeTable();
        }

        if(current != null) {
            return Optional.ofNullable(current.getValue());
        }
        return Optional.empty();
    }

    public Optional<R> get(T key) {
        int index = getIndex(key);
        Entry<T, R> entry = table[index];
        if(entry != null)
            return Optional.ofNullable(entry.getValue());
        return Optional.empty();
    }

    public boolean has(T key) {
        return get(key).isPresent();
    }

    public int getSize() {
        return size;
    }

    protected void resizeTable() {
        internalSize = size = 0;
        Entry<T, R>[] tmp = table;
        table = new Entry[(int) ((table.length /fillRatio)* 2)];
        for(Entry<T, R> entry : tmp){
            if(entry != null) {
                put(entry.getKey(), entry.getValue());
            }
        }
    }

    protected int getIndex(T key) {
        int hash = key.hashCode();
        int index = (((hash % table.length) + table.length) % table.length);
        while(table[index] != null && (table[index].getKey().hashCode() != hash || !table[index].getKey().equals(key))) {
            if(++index == table.length) {
                index = 0;
            }
        }
        return index;
    }

    public static final class Entry <T, R> {
        private final T key;
        private final R value;

        public Entry(T key, R value) {
            this.key = key;
            this.value = value;
        }

        public T getKey() {
            return key;
        }

        public R getValue() {
            return value;
        }
    }

}

```
### BinaryHeap
```

```
### Trie
```
public class Trie {

    private static final int ALPHABET_SIZE = 26;
    private final Trie[] children = new Trie[ALPHABET_SIZE];
    private boolean isWordEnd;

    public void add(String s) {
        if(s.length() > 0) {
            int index = getIndex(s.charAt(0));
            if(children[index] == null) {
                children[index] = new Trie();
            }
            Trie child = children[index];
            child.add(s.substring(1));
        } else {
            isWordEnd = true;
        }
    }

    public boolean has(String s) {
        if(s.length() == 0)
            return isWordEnd;
        int index = getIndex(s.charAt(0));
        if(children[index] == null)
            return false;
        return children[index].has(s.substring(1));
    }

    private int getIndex(char c) {
        return c - 'a';
    }
}
```
### ArrayList
```
import java.util.Arrays;
import java.util.Optional;

@SuppressWarnings("unchecked")
public class ArrayList<T> {

    private T[] list;
    private int size;

    public ArrayList() {
        this(2);
    }

    public ArrayList(int initialCapacity) {
        this.list = (T[]) new Object[initialCapacity];
    }

    public void add(T value) {
        list[size] = value;
        if(++size == list.length) {
            list = Arrays.copyOf(list, list.length * 2);
        }
    }

    public T get(int index) {
        return list[index];
    }

    public Optional<Integer> indexOf(T value) {
        for(int i = 0; i < size; i++) {
            if(list[i].equals(value)) {
                return Optional.of(i);
            }
        }
        return Optional.empty();
    }

    public boolean has(T value) {
        return indexOf(value).isPresent();
    }

    public int getSize() {
        return size;
    }

}
```

## Object Oriented Programming - Java

### Class
```
public class Frog {

}
```
### Instance/Object
```
Frog fred = new Frog();
```
### Method
```
public class Frog {
  public int getColor() {
    return 0xff2dbd3a;
  }
}
```
### Method Override / Overload
```
public abstract class Reptile {
  public abstract int getColor(int alpha);
}

public class Frog extends Reptile {
  public int getColor() {
    return getColor(0xff);
  }

  @Override
  public int getColor(int alpha) {
    return (alpha << 24) | 0x002dbd3a;
  }
}
```
### Static method
```
public class Main {
  public static void main(String[] args) {

  }
}
```
### Constructor
```
public class Frog extends Reptile {
  public Frog() {}
  // ...
}
```
###


superclass or base class
subclass or derived class
inheritance
encapsulation
multiple inheritance (and give an example)
delegation/forwarding
composition/aggregation
abstract class
interface/protocol (and different from abstract class)
method overriding
method overloading (and difference from overriding)
polymorphism (without resorting to examples)
is-a versus has-a relationships (with examples)
method signatures (what's included in one)
method visibility (e.g. public/private/other)


fourier transform
