+++
Categories = ["Development", "Java", "CS"]
Description = "Implementations of commonly discussed data structures and algorithms"
Tags = ["java", "algorithms", "data", "structures", "big-o"]
date = "2016-02-23T05:27:09-06:00"
menu = "main"
title = "Basic Data Structures and Algorithms"

+++

#### Description

This post contains the implementations, in Java 8, of a few data structures and algorithms commonly discussed within computer science.

The full source code for this post can be found [here](https://github.com/davityle/cs-basics-java)

The tests for the data structures and algorithms can be found [here](https://github.com/davityle/cs-basics-java/tree/master/test/com/github/davityle/csbasics)

[Hash Map]({{< relref "computer-science-basics.md#hash-map" >}})

[Graph]({{< relref "computer-science-basics.md#graph" >}})

[Array List]({{< relref "computer-science-basics.md#array-list" >}})

[Linked List]({{< relref "computer-science-basics.md#linked-list" >}})

[Tree]({{< relref "computer-science-basics.md#tree" >}})

[Trie]({{< relref "computer-science-basics.md#trie" >}})

[Quick Sort]({{< relref "computer-science-basics.md#quick-sort" >}})

[Merge Sort]({{< relref "computer-science-basics.md#merge-sort" >}})

[A*]({{< relref "computer-science-basics.md#a" >}})

[Convex Hull]({{< relref "computer-science-basics.md#convex-hull" >}})
<!--more-->
#### Data Structures

##### Hash Map

A hash map is an array backed data structure that stores key-value pairs. It finds the index of the key-value pair using the hash code of the key modulus the length of the hash array. If two different keys have the same index a collision algorithm is used to correctly store the values. Many hash map implementations use buckets for hash collisions. This hash map does not. Each unique key has it's own index.

{{< highlight java>}}
package com.github.davityle.csbasics.data.map;

import com.github.davityle.csbasics.util.Maybe;

@SuppressWarnings("unchecked")
public abstract class HashMap<T, R> {

    private static final int MIN_CAPACITY = 2;
    protected Entry<T, R>[] table;
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

    public Maybe<R> put(T key, R value) {
        int index = getIndex(key);
        Entry<T, R> current = table[index];
        table[index] = new Entry<>(key, value);

        if (value == null && current != null && current.getValue() != null) {
            size--;
        } else if (value != null && (current == null || current.getValue() == null)) {
            size++;
        }

        if (current == null && ++internalSize >= ((table.length * fillRatio) - 1)) {
            resizeTable();
        }

        if (current != null) {
            return Maybe.ofNullable(current.getValue());
        }
        return Maybe.empty();
    }

    public Maybe<R> get(T key) {
        int index = getIndex(key);
        Entry<T, R> entry = table[index];
        if (entry != null)
            return Maybe.ofNullable(entry.getValue());
        return Maybe.empty();
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
        table = new Entry[(int) ((table.length / fillRatio) * 2)];
        for (Entry<T, R> entry : tmp) {
            if (entry != null) {
                put(entry.getKey(), entry.getValue());
            }
        }
    }

    protected int modLength(int hash) {
        return (((hash % table.length) + table.length) % table.length);
    }

    protected boolean collision(int index, int hash, T key) {
        return table[index] != null && (table[index].getKey().hashCode() != hash || !table[index].getKey().equals(key));
    }

    protected abstract int getIndex(T key);

    public static final class Entry<T, R> {
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

{{< /highlight >}}
###### Hash Map Collision Algorithms
{{< highlight java >}}
package com.github.davityle.csbasics.data.map;

public class LinearProbingHashMap<T, R> extends HashMap<T, R> {
    @Override
    protected int getIndex(T key) {
        int hash = key.hashCode();
        int index = modLength(hash);
        while (collision(index, hash, key)) {
            if (++index == table.length) {
                index = 0;
            }
        }
        return index;
    }
}
{{< /highlight >}}
{{< highlight java >}}
package com.github.davityle.csbasics.data.map;

public class QuadraticProbingHashMap<T, R> extends HashMap<T, R> {

    @Override
    protected int getIndex(T key) {
        int hash = key.hashCode();
        int index = modLength(hash);
        int count = 0;
        while (collision(index, hash, key)) {
            count++;
            index = (index + count * count) % table.length;
        }
        return index;
    }
}
{{< /highlight >}}

Other hashmap implementations:

- [Java/C++](http://www.algolist.net/Data_structures/Hash_table/Simple_example)
- [Haskell](https://github.com/gregorycollins/hashtables)
- [Python](http://www.saschaschnepp.net/2013/12/01/python_hashtable/)

##### Graph

There are several ways to implement graphs in memory. Two of the most popular are adjacency lists and adjacency matrices.

###### Graph Interface
{{< highlight java >}}
package com.github.davityle.csbasics.data.graph;

import java.util.Collection;
import java.util.List;
import java.util.Set;

public interface Graph<T, V extends Graph.Vertex<T>> {

    void addVertex(V vertex);
    void addVertices(Collection<V> vertices);
    Set<V> getVertices();
    void addEdge(V a, V b);
    void addEdges(V vertex, Collection<V> vertices);
    List<V> getAdjacentVertices(V vertex);

    class Vertex<T> {
        public final double x, y;
        public final T value;

        public Vertex(double x, double y, T value) {
            this.x = x;
            this.y = y;
            this.value = value;
        }

        @Override
        public boolean equals(Object obj) {
            if(!(obj instanceof Vertex))
                return false;
            Vertex vert = (Vertex) obj;
            return x == vert.x && y == vert.y;
        }

        @Override
        public int hashCode() {
            return (int) (x * 7 + y * 13);
        }

        @Override
        public String toString() {
            return String.format("(%.2f, %.2f) - %s", x, y, value);
        }
    }

    class Edge<T> {
        public final Graph.Vertex<T> a, b;

        public Edge(Graph.Vertex<T> a, Graph.Vertex<T> b) {
            this.a = a;
            this.b = b;
        }

        @Override
        public String toString() {
            return "{" + a + ":" + b + "}";
        }
    }
}
{{< /highlight >}}
###### Adjacency List Graph
{{< highlight java >}}
package com.github.davityle.csbasics.data.graph;

import java.util.*;

public class ListGraph<T> implements Graph<T, ListVertex<T>> {

    Set<ListVertex<T>> vertices = new HashSet<>();

    @Override
    public void addVertex(ListVertex<T> vertex) {
        this.vertices.add(vertex);
    }

    @Override
    public void addVertices(Collection<ListVertex<T>> vertices) {
        this.vertices.addAll(vertices);
    }

    @Override
    public Set<ListVertex<T>> getVertices() {
        return vertices;
    }

    @Override
    public void addEdge(ListVertex<T> a, ListVertex<T> b) {
        a.addAdjacentVertex(b);
    }

    @Override
    public void addEdges(ListVertex<T> vertex, Collection<ListVertex<T>> vertices) {
        vertex.addAdjacentVertices(vertices);
    }

    @Override
    public List<ListVertex<T>> getAdjacentVertices(ListVertex<T> vertex) {
        return vertex.getAdjacentVertices();
    }


}
{{< /highlight >}}
{{< highlight java >}}
package com.github.davityle.csbasics.data.graph;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;

public class ListVertex<T> extends Graph.Vertex<T> {
    private final List<ListVertex<T>> adjacent = new ArrayList<>();

    public ListVertex(double x, double y, T value) {
        super(x, y, value);
    }

    void addAdjacentVertex(ListVertex<T> listVertex) {
        adjacent.add(listVertex);
    }

    void addAdjacentVertices(Collection<ListVertex<T>> adjVertices) {
        adjacent.addAll(adjVertices);
    }

    List<ListVertex<T>> getAdjacentVertices() {
        return adjacent;
    }
}

{{< /highlight >}}
###### Adjacency Matrix Graph
{{< highlight java >}}
package com.github.davityle.csbasics.data.graph;


import java.util.*;

@SuppressWarnings("unchecked")
public class MatrixGraph<T> implements Graph<T, Graph.Vertex<T>> {

    private HashMap<Vertex<T>, Integer> verticesIndexes;
    private boolean[][] adjencencyMatrix;
    private Vertex<T>[] vertices;
    private int size;

    public MatrixGraph() {
        this(2);
    }

    public MatrixGraph(int initialSize) {
        adjencencyMatrix = new boolean[initialSize][initialSize];
        vertices = new Vertex[initialSize];
        verticesIndexes = new HashMap<>();
    }

    public Graph.Vertex<T> getVertex(int index) {
        return vertices[index];
    }

    @Override
    public void addVertex(Vertex<T> vertex) {
        verticesIndexes.put(vertex, size);
        vertices[size++] = vertex;
        checkResize();
    }

    private void checkResize() {
        if(size >= adjencencyMatrix.length) {
            resize();
        }
    }

    private void resize() {
        boolean[][] tmp = adjencencyMatrix;
        int length = adjencencyMatrix.length * 2;
        adjencencyMatrix = new boolean[length][length];
        for(int i = 0; i < tmp.length; i++) {
            System.arraycopy(tmp[i], 0, adjencencyMatrix[i], 0, tmp.length);
        }
        Vertex<T>[] vertexTmp = vertices;
        vertices = new Vertex[length];
        System.arraycopy(vertexTmp, 0, vertices, 0, vertexTmp.length);
    }

    @Override
    public void addVertices(Collection<Vertex<T>> vertices) {
        vertices.forEach(this::addVertex);
    }

    @Override
    public Set<Vertex<T>> getVertices() {
        return verticesIndexes.keySet();
    }

    @Override
    public void addEdge(Vertex<T> a, Vertex<T> b) {
        Integer idA = verticesIndexes.get(a);
        Integer idB = verticesIndexes.get(b);
        if(idA == null || idB == null)
            throw new IllegalArgumentException("Vertices must be in graph to add an edge");
        adjencencyMatrix[idA][idB] = true;
        adjencencyMatrix[idB][idA] = true;
    }

    @Override
    public void addEdges(Vertex<T> vertex, Collection<Vertex<T>> vertices) {
        vertices.forEach(v -> addEdge(vertex, v));
    }

    @Override
    public List<Vertex<T>> getAdjacentVertices(Vertex<T> vertex) {
        Integer id = verticesIndexes.get(vertex);
        if(id == null)
            throw new IllegalArgumentException(String.format("Vertex %s was not found", vertex));
        List<Vertex<T>> vertices = new ArrayList<>();

        for(int i = 0; i < size; i++) {
            if(adjencencyMatrix[id][i]) {
                vertices.add(this.vertices[i]);
            }
        }

        return vertices;
    }
}

{{< /highlight >}}

Other graph implementations:

- [Java](https://www.cs.duke.edu/courses/cps100e/fall10/class/11_Bacon/code/Graph.html)
- [C++](http://www.sanfoundry.com/cpp-program-implement-adjacency-list/)
- [Python](http://www.python-course.eu/graphs_python.php)
- [Scala](https://github.com/archie/scalacaster/blob/master/src/graph/InductiveGraph.scala)

##### Array List

An array list is a list of arbitrary size backed by a self resizing array.

{{< highlight java >}}
package com.github.davityle.csbasics.data.list;

import java.util.Arrays;
import java.util.Optional;

@SuppressWarnings("unchecked")
public class ArrayList<T> implements List<T> {

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
        if (++size == list.length) {
            list = Arrays.copyOf(list, list.length * 2);
        }
    }

    public T get(int index) {
        return list[index];
    }

    public T remove(int index) {
        T value = list[index];
        System.arraycopy(list, index + 1, list, index, size - (index + 1));
        size--;
        return value;
    }

    public OptionalInt indexOf(T value) {
        for (int i = 0; i < size; i++) {
            if (list[i] != null && list[i].equals(value)) {
                return OptionalInt.of(i);
            }
        }
        return OptionalInt.empty();
    }

    public boolean has(T value) {
        return indexOf(value).isPresent();
    }

    public int getSize() {
        return size;
    }

}
{{< /highlight >}}

Other array list implementations:

- [Java](http://developer.classpath.org/doc/java/util/ArrayList-source.html)
- [C++](https://gist.github.com/daeltar/90951)

##### Linked List

An linked list is a list of arbitrary size that is stored using nodes with pointers to their neighbor nodes.

{{< highlight java >}}
package com.github.davityle.csbasics.data.list;

import java.util.Optional;

public class LinkedList<T> implements Queue<T>, Stack<T>, List<T> {

    private int size;
    private final Node<T> headNode, tailNode;

    public LinkedList() {
        this.headNode = new Node<>(null);
        this.tailNode = new Node<>(null);
        tailNode.previous = headNode;
    }

    public void add(T value) {
        Node<T> tail = tailNode.previous;
        Node<T> newNode = new Node<>(value);

        tail.next = newNode;
        newNode.previous = tail;
        tailNode.previous = newNode;

        size++;
    }

    public T poll() {
        return remove(0);
    }

    public void push(T value) {
        add(value);
    }

    public T pop() {
        T value = tailNode.previous.value;
        tailNode.previous = tailNode.previous.previous;
        tailNode.previous.next = null;
        size--;
        return value;
    }

    public T get(int index) {
        Node<T> n = headNode.next;
        for (int i = 0; i < index; i++) {
            n = n.next;
        }
        return n.value;
    }

    public T remove(int index) {
        if(index == size - 1) {
            return pop();
        }
        Node<T> left = headNode, n = headNode.next;
        for (int i = 0; i < index; i++) {
            left = n;
            n = n.next;
        }
        left.next = n.next;
        left.next.previous = left;
        size--;
        return n.value;
    }

    public OptionalInt indexOf(T value) {
        if(value == null)
            throw new IllegalArgumentException("value must not be null");
        Node<T> n = headNode.next;
        int i = 0;
        while (n != null) {
            if (value.equals(n.value))
                return OptionalInt.of(i);
            i++;
            n = n.next;
        }
        return OptionalInt.empty();
    }

    public boolean has(T value) {
        return indexOf(value).isPresent();
    }

    public int getSize() {
        return size;
    }

    private static class Node<T> {
        private T value;

        public Node(T value) {
            this.value = value;
        }

        private Node<T> next;
        private Node<T> previous;
    }

}
{{< /highlight >}}

Other linked list implementations:

- [Java](https://www.cs.cmu.edu/~adamchik/15-121/lectures/Linked%20Lists/code/LinkedList.java)
- [C++](http://www.sanfoundry.com/cpp-program-implement-single-linked-list/)
- [Haskell](https://gist.github.com/akoskovacs/5074344)
- [Python](https://www.codefellows.org/blog/implementing-a-singly-linked-list-in-python)

##### Tree

You can find an explanation for trees [here](http://www.cs.cmu.edu/~clo/www/CMU/DataStructures/Lessons/lesson4_1.htm).
This tree has both [depth first](https://en.wikipedia.org/wiki/Depth-first_search) and [breadth first](https://en.wikipedia.org/wiki/Breadth-first_search) iterations.
This implementation does not allow duplicates as determined by `Object#hashcode()` and `Object#equals()`.

{{< highlight java >}}
package com.github.davityle.csbasics.data.tree;


import java.util.*;
import java.util.stream.Collectors;

public class TreeNode<T> {

    private final HashMap<T, TreeNode<T>> children = new HashMap<>();
    private final T value;
    private TreeNode<T> parent;


    public TreeNode() {
        this(null);
    }

    public TreeNode(T value) {
        this.value = value;
    }

    public Optional<TreeNode<T>> getNode(T child) {
        return Optional.ofNullable(children.get(child));
    }

    public TreeNode<T> getNodeOrThrow(T child) {
        return getNode(child).get();
    }

    public boolean has(T s) {
        return getNode(s).isPresent();
    }

    public TreeNode<T> add(T child) {
        return add(new TreeNode<>(child));
    }

    public TreeNode<T> add(TreeNode<T> child) {
        child.setParent(this);
        children.put(child.value, child);
        return child;
    }

    public void add(Collection<T> values) {
        values.forEach(this::add);
    }

    public void setParent(TreeNode<T> parent) {
        this.parent = parent;
    }

    public HashMap<T, TreeNode<T>> getChildren() {
        return children;
    }

    public TreeNode<T> getParent() {
        return parent;
    }

    public T getValue() {
        return value;
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof TreeNode))
            return false;
        TreeNode node = (TreeNode) obj;
        if (node.value == value)
            return true;
        if (node.value == null)
            return false;
        return node.value.equals(value);
    }

    @Override
    public int hashCode() {
        return value != null ? value.hashCode() : super.hashCode();
    }

    public Iterator<T> breadthIterator() {
        return new BreadthIterator();
    }

    public Iterator<T> depthIterator() {
        return new DepthIterator();
    }

    private class BreadthIterator implements Iterator<T> {
        final Queue<TreeNode<T>> queue = new LinkedList<>();

        private BreadthIterator() {
            queue.addAll(TreeNode.this.children.values());
        }

        @Override
        public boolean hasNext() {
            return !queue.isEmpty();
        }

        @Override
        public T next() {
            TreeNode<T> node = queue.poll();
            queue.addAll(node.children.values());
            return node.value;
        }
    }

    private class DepthIterator implements Iterator<T> {
        final Stack<TreeNode<T>> stack = new Stack<>();

        private DepthIterator() {
            addChildren(TreeNode.this);
        }

        @Override
        public boolean hasNext() {
            return !stack.isEmpty();
        }

        @Override
        public T next() {
            TreeNode<T> node = stack.pop();
            addChildren(node);
            return node.value;
        }

        private void addChildren(TreeNode<T> node) {
            node.children.values().stream()
                    .collect(Collectors.toCollection(LinkedList::new))
                    .descendingIterator()
                    .forEachRemaining(stack::push);
        }
    }
}
{{< /highlight >}}

##### Binary Heap

A binary heap is a sorted tree data structure. When a value is inserted it is bubbled down to it's correct position. This binary heap is backed by an array.

{{< highlight java >}}
package com.github.davityle.csbasics.data.heap;

import java.util.Arrays;
import java.util.Comparator;

@SuppressWarnings("unchecked")
public class BinaryHeap<T extends Comparable<T>> {

    private final Comparator<T> comparator;
    private T[] heap;
    private int size;

    public BinaryHeap() {
        this(2);
    }

    public BinaryHeap(int initialCapacity) {
        this(initialCapacity, Comparable::compareTo);
    }

    public BinaryHeap(Comparator<T> comparator) {
        this(2, comparator);
    }

    public BinaryHeap(int initialCapacity, Comparator<T> comparator) {
        this.heap = (T[]) new Comparable[initialCapacity];
        this.comparator = comparator;
    }

    public T peek() {
        return heap[0];
    }

    public int getSize() {
        return size;
    }

    public boolean isEmpty() {
        return size == 0;
    }

    public void add(T value) {
        int index = size, parentIndex;
        // bubble up
        while(index > 0 && comparator.compare(value, heap[(parentIndex = parentIndex(index))]) < 0) {
            heap[index] = heap[parentIndex];
            index = parentIndex;
        }
        heap[index] = value;
        // resize if needed
        if (++size == heap.length) {
            heap = Arrays.copyOf(heap, heap.length * 2);
        }
    }

    public T poll() {
        T value = heap[0];
        T tmp = heap[--size];

        int index = 0, childIndex;
        // bubble down
        while ((childIndex = lesserChildIndex(index)) != -1 && comparator.compare(heap[childIndex], tmp) < 0) {
            heap[index] = heap[childIndex];
            index = childIndex;
        }
        heap[index] = tmp;

        return value;
    }

    private int lesserChildIndex(int ind) {
        int fstChild = kthChildIndex(ind, 1);
        if(fstChild >= size)
            return -1;
        int sndChild = kthChildIndex(ind, 2);
        boolean second = sndChild < size && comparator.compare(heap[sndChild], heap[fstChild]) < 0;
        return second ? sndChild : fstChild;
    }

    private int parentIndex(int index) {
        return (index - 1)/2;
    }

    private int kthChildIndex(int index, int k) {
        return 2*index + k;
    }
}
{{< /highlight >}}


##### Trie

A trie is a special subclass of Tree that is designed to be an extremely quick dictionary lookup. This implementation only supports case insensitive a-z characters.

{{< highlight java >}}
package com.github.davityle.csbasics.data.tree;

public class Trie {

    private static final int ALPHABET_SIZE = 26;
    private final Trie[] children = new Trie[ALPHABET_SIZE];
    private boolean isWordEnd;

    public void add(String s) {
        if (s.length() > 0) {
            int index = getIndex(s.charAt(0));
            if (children[index] == null) {
                children[index] = new Trie();
            }
            Trie child = children[index];
            child.add(s.substring(1));
        } else {
            isWordEnd = true;
        }
    }

    public boolean has(String s) {
        if (s.length() == 0)
            return isWordEnd;
        int index = getIndex(s.charAt(0));
        return children[index] != null && children[index].has(s.substring(1));
    }

    private int getIndex(char c) {
        return Character.toLowerCase(c) - 'a';
    }
}

{{< /highlight >}}
A trie that supports all characters can easily be implemented using the TreeNode.
{{< highlight java >}}
package com.github.davityle.csbasics.data.tree;


public class TreeTrie {

    final TreeNode<Character> tree = new TreeNode<>();

    public void add(String s) {
        add(s, tree);
    }

    private void add(String s, TreeNode<Character> node) {
        if (s.length() > 0) {
            add(s.substring(1), node.getNode(s.charAt(0)).orElseGet(() -> {
                TreeNode<Character> newNode = new TreeNode<>(s.charAt(0));
                node.add(newNode);
                return newNode;
            }));
        } else {
            node.add('\0');
        }
    }

    public boolean has(String s) {
        return has(s, tree);
    }

    private boolean has(String s, TreeNode<Character> node) {
        if (s.length() == 0)
            return node.has('\0');
        return node.getNode(s.charAt(0)).map(child -> has(s.substring(1), child)).orElseGet(() -> false);
    }

}

{{< /highlight >}}

#### Algorithms

##### Quick Sort

Quick sort takes an array and sorts it in average case O(n*log(n)) time, O(n^2) worst case.

{{< highlight java >}}
package com.github.davityle.csbasics.algorithm.sort;

import java.util.Random;

public class QuickSort {
    QuickSort() {
    }

    private static final Random rand = new Random();

    public static <T extends Comparable<T>> void sort(T[] array) {
        sort(array, 0, array.length - 1);
    }

    public static <T extends Comparable<T>> void sort(T[] array, int bottom, int top) {
        if (bottom < top) {
            int pivot = pivot(array, bottom, top);
            sort(array, bottom, pivot);
            sort(array, pivot + 1, top);
        }
    }

    private static <T extends Comparable<T>> int pivot(T[] array, int bottom, int top) {
        int pivot = bottom + rand.nextInt(top - bottom);
        int leftIndex = bottom - 1;
        int rightIndex = top + 1;
        T value = array[pivot];
        while (true) {
            do {
                leftIndex++;
            } while (leftIndex < rightIndex && array[leftIndex].compareTo(value) < 0);

            do {
                rightIndex--;
            } while (leftIndex < rightIndex && array[rightIndex].compareTo(value) > 0);

            if (leftIndex < rightIndex)
                swap(array, leftIndex, rightIndex);
            else
                break;
        }
        return pivot;
    }

    private static <T extends Comparable<T>> void swap(T[] array, int leftIndex, int rightIndex) {
        T tmp = array[leftIndex];
        array[leftIndex] = array[rightIndex];
        array[rightIndex] = tmp;
    }

}
{{< /highlight >}}

##### Merge Sort

Merge sort is generally slower than Quick sort but it's worst case is O(n * log(n)).

{{< highlight java >}}
package com.github.davityle.csbasics.algorithm.sort;

public class MergeSort {

    MergeSort() {
    }

    public static <T extends Comparable<T>> void sort(T[] array) {
        sort(array, 0, array.length, (T[]) new Comparable[array.length]);
    }

    public static <T extends Comparable<T>> void sort(T[] array, int min, int max, T[] tmp) {

        if (max - min < 2)
            return;

        int mid = (max + min) / 2;

        sort(array, min, mid, tmp);
        sort(array, mid, max, tmp);

        int left = min, right = mid;
        for (int i = min; i < max; i++) {
            if (left < mid && (right >= max || array[left].compareTo(array[right]) <= 0)) {
                tmp[i] = array[left];
                left = left + 1;
            } else {
                tmp[i] = array[right];
                right = right + 1;
            }
        }

        System.arraycopy(tmp, min, array, min, max - min);
    }
}
{{< /highlight >}}
##### A*

A* is an algorithm for finding the shortest path between two nodes in a graph.

{{< highlight java >}}
package com.github.davityle.csbasics.algorithm.graph;

import com.github.davityle.csbasics.data.graph.Graph;
import com.github.davityle.csbasics.data.heap.BinaryHeap;

import java.util.*;

public class AStar {

    private static final class Node<T, V extends Graph.Vertex<T>> implements Comparable<Node<T, V>> {
        private final V vertex;
        private final double distance;
        private final double estimatedLength;
        private final Node<T, V> previous;

        private Node(V vertex, double distance, double heuristic, Node<T, V> previous) {
            this.vertex = vertex;
            this.distance = distance;
            this.estimatedLength = distance + heuristic;
            this.previous = previous;
        }

        @Override
        public int hashCode() {
            return vertex.hashCode();
        }

        @Override
        public boolean equals(Object obj) {
            if (!(obj instanceof Node))
                return false;
            Node n = (Node) obj;
            return vertex.equals(n.vertex);
        }

        @Override
        public int compareTo(Node<T, V> n) {
            if (estimatedLength > n.estimatedLength)
                return 1;
            if (estimatedLength < n.estimatedLength)
                return -1;
            return 0;
        }

        @Override
        public String toString() {
            return String.format("%.4f - %s", estimatedLength, vertex);
        }
    }


    public static <T, V extends Graph.Vertex<T>> Optional<List<V>> shortestRoute(V start, V end, Graph<T, V> graph) {
        BinaryHeap<Node<T, V>> priorityQueue = new BinaryHeap<>();
        Set<Node<T, V>> visited = new HashSet<>();

        priorityQueue.add(new Node<>(start, 0, 0, null));
        while (!priorityQueue.isEmpty()) {
            final Node<T, V> node = priorityQueue.poll();
            if (visited.add(node)) {
                for (V vertex : graph.getAdjacentVertices(node.vertex)) {

                    Node<T, V> nextNode = new Node<>(vertex, node.distance + length(node.vertex, vertex), length(vertex, end), node);

                    if (nextNode.vertex.equals(end)) {
                        return Optional.of(backTrace(nextNode));
                    }

                    if (!visited.contains(nextNode)) {
                        priorityQueue.add(nextNode);
                    }
                }
            }
        }

        return Optional.empty();
    }

    private static <T, V extends Graph.Vertex<T>> List<V> backTrace(Node<T, V> node) {
        List<V> route = new LinkedList<>();
        while (node != null) {
            route.add(0, node.vertex);
            node = node.previous;
        }
        return route;
    }

    private static <T> double length(Graph.Vertex<T> v1, Graph.Vertex<T> v2) {
        return Math.sqrt(Math.pow(v1.x - v2.x, 2) + Math.pow(v1.y - v2.y, 2));
    }

}
{{< /highlight >}}

A visualization of the shortest path between two points found using A*.

![A* Visualization](/images/a_star.png)


##### Convex Hull

Convex hull is an algorithm that finds the outermost path around a set of points. This implementation of Convex Hull is loosely based on Quick Hull but is ultimately my own brain child. It is faster than Quick Hull but that is the craziest claim that I'll make.

{{< highlight java >}}
package com.github.davityle.csbasics.algorithm.graph;

import com.github.davityle.csbasics.data.graph.Graph;

import java.util.*;
import java.util.stream.Collectors;

public class ConvexHull {

    public static <T> List<Graph.Edge<T>> outerMostPath(Graph<T, Graph.Vertex<T>> graph) {
        return outerMostPath(new ArrayList<>(graph.getVertices()));
    }

    public static <T> List<Graph.Edge<T>> outerMostPath(List<Graph.Vertex<T>> list) {
        if(list.size() == 0)
            return Collections.emptyList();

        Comparator<Graph.Vertex<T>> xComparator = (v1, v2) -> {
            if (v1.x > v2.x)
                return -1;
            if (v1.x < v2.x)
                return 1;
            return 0;
        };

        Comparator<Graph.Vertex<T>> yComparator = (v1, v2) -> {
            if (v1.y > v2.y)
                return -1;
            if (v1.y < v2.y)
                return 1;
            return 0;
        };

        Graph.Vertex<T> left = list.stream().min(xComparator).get();
        Graph.Vertex<T> right = list.stream().max(xComparator).get();

        Graph.Vertex<T> bottom = list.stream().min(yComparator).get();
        Graph.Vertex<T> top = list.stream().max(yComparator).get();

        List<Graph.Edge<T>> edges = new ArrayList<>();

        edges.addAll(quickHull(new Graph.Edge<>(left, top), list));
        edges.addAll(quickHull(new Graph.Edge<>(top, right), list));
        edges.addAll(quickHull(new Graph.Edge<>(right, bottom), list));
        edges.addAll(quickHull(new Graph.Edge<>(bottom, left), list));

        return edges;
    }

    private static <T> List<Graph.Edge<T>> quickHull(Graph.Edge<T> edge, List<Graph.Vertex<T>> points) {
        List<Graph.Vertex<T>> filtered = points.stream().filter(point -> distanceFromEdge(edge, point) > 0).collect(Collectors.toList());

        return filtered.stream()
                .min((p1, p2) -> {
                    double p1Distance = distanceFromEdge(edge, p1);
                    double p2Distance = distanceFromEdge(edge, p2);
                    if (p1Distance > p2Distance)
                        return -1;
                    if (p1Distance < p2Distance)
                        return 1;
                    return 0;
                })
                .map(p -> {
                    List<Graph.Edge<T>> edges = new ArrayList<>();
                    edges.addAll(quickHull(new Graph.Edge<>(edge.a, p), filtered));
                    edges.addAll(quickHull(new Graph.Edge<>(p, edge.b), filtered));
                    return edges;
                })
                .orElse(Collections.singletonList(edge));
    }

    private static <T> double distanceFromEdge(Graph.Edge<T> l, Graph.Vertex<T> c) {
        return (l.b.x - l.a.x) * (c.y - l.a.y) - (l.b.y - l.a.y) * (c.x - l.a.x);
    }

}
{{< /highlight >}}

![Convex Hull Visualization](/images/convex_hull.png)

-----

If you'd like to see usages of any of these data structures or algorithms just check out the [tests](https://github.com/davityle/cs-basics-java/tree/master/test/com/github/davityle/csbasics)
