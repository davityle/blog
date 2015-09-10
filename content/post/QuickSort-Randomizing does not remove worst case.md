+++
date = "2015-01-31T10:01:16-07:00"
title = "QuickSort-Randomizing does not remove worst case"
Categories = ["algorithms"]
Tags = ["algorithms", "quicksort", "java", "big-o"]
+++

## Problem

I've read in a few places that Randomizing the pivot point removes the worst case of O(n<span style="position: relative; bottom: 1ex; font-size: 80%;">2</span>) from the quicksort algorithm. [Wikipedia](http://en.wikipedia.org/wiki/Quicksort#Analysis_of_randomized_quicksort) is one of those places. Other places write the whole story but don't show any proof. The whole story is that randomizing the pivot point does not, necessarily, remove the worst case. It's easy to show that this is the case. This article assumes basic knowledge of the quicksort algorithm and big O notation.

The worst case in quick sort happens when each pivot point always contains the next lowest, or always contains the next highest, number in the sequence.

For example. <br> You have an unsorted array

```
array = {44,88,7,2,1,999,1040,23,123,89,2009}
```
and you want to traverse this array using random pivots. Lets just say that the random number generator produces theses numbers.
```
4,3,2,7,4,7,9,8,8,9
```
Those look random enough. In fact, I would be perfectly comfortable sorting my array with those random numbers just off of the first glance.

Look at what actually happens when you sort the array using those pivots.

```
pivot: 4, array[4] == 1
```
the pivot index is 4, the value at index 4 is 1. Notice that 1 is the lowest number in the array. After the first iteration you end up with this array
```
{1,88,7,2,44,999,1040,23,123,89,2009}
```
I'm just going to run through the rest of the iterations here. Try not to get lost.
```
pivot: 3	 value: 2
array = {1,88,7,2,44,999,1040,23,123,89,2009}

pivot: 2	 value: 7
array = {1,2,7,88,44,999,1040,23,123,89,2009}

pivot: 7	 value: 23
array = {1,2,7,88,44,999,1040,23,123,89,2009}

pivot: 4	 value: 44
array = {1,2,7,23,44,999,1040,88,123,89,2009}

pivot: 7	 value: 88
{1,2,7,23,44,999,1040,88,123,89,2009}

pivot: 9	 value: 89
{1,2,7,23,44,88,1040,999,123,89,2009}

pivot: 8	 value: 123
{1,2,7,23,44,88,89,999,123,1040,2009}

pivot: 8	 value: 999
{1,2,7,23,44,88,89,123,999,1040,2009}

pivot: 9	 value: 1040
{1,2,7,23,44,88,89,123,999,1040,2009}
```
<!--more-->
Hurray! The list is sorted. What do you notice about the number of iterations it took though? What do you notice about the values of the numbers?

Notice that it took about n iterations , and each iteration handled every number that wasn't already sorted. This results in a n operations over n iterations, or O(n<span style="position: relative; bottom: 1ex; font-size: 80%;">2</span>), operation.

Even though this array is sorted using a random pivot it still hit the worst possible case.

## Proof

What if you had a way to generate this worst case scenario? Luckily, I was able to reverse engineer the quicksort algorithm and produce a shuffle that always shuffles the array into the worst case scenario for quicksort (if they have the same random seed). The simple algorithm is defined below.

```
/**
 * This class creates the worst case scenario for a quick sort
 */
public class QuickShuffle {
	/**
	* assumes a sorted array is passed in
	*/
	public static void quickShuffle(int[] a, Random rand){		
		int length = a.length - 1;
		int[] indexes= new int[length];
		for(int index = 0; index < length; index++){
			int w = index + rand.nextInt(length - index);
			indexes[index] = w;
		}
		for(int index = indexes.length - 1; index >= 0; index--){
			swap(a, index, indexes[index]);
		}
	}

	private static void swap(int[] a, int index1, int index2) {
		int temp = a[index1];
		a[index1] = a[index2];
		a[index2] = temp;
	}
}
```
Then with a quicksort that looks like this
```
public class QuickSort {

	private Random rand;

	private QuickSort(Random rand){
		this.rand = rand;
	}

	public static void quickSort(int[] a, Random rand){
		new QuickSort(rand).quickSort(a, 0, a.length - 1);
	}

	private QuickSort quickSort(int[] a, int bottomIndex, int topIndex){
		if(bottomIndex<topIndex){
			int pivot =switchPivot(a,bottomIndex,topIndex);
			quickSort(a,bottomIndex,pivot);
			quickSort(a,pivot+1,topIndex);
		}
		return this;
	}

	private int switchPivot(int[] a, int bottomIndex, int topIndex) {
		int index = bottomIndex + rand.nextInt(topIndex - bottomIndex);
		int value = a[index];
		int bottomSearch = bottomIndex-1 ;
		int topSearch = topIndex+1 ;

		while (true) {
		    do{
		    	bottomSearch++;
		    }while ( bottomSearch < topIndex && a[bottomSearch] < value);

		    do{
		    	topSearch--;
		    }while (topSearch > bottomIndex && a[topSearch] > value);

		    if (bottomSearch < topSearch)
		    	swap(a, bottomSearch, topSearch);
		    else
		    	return topSearch;
		}
	}

	private void swap(int[] a, int index1, int index2) {
		int temp = a[index1];
		a[index1] = a[index2];
		a[index2] = temp;
	}
}
```
We can do some profiling and check to see if their is any validity to this worst case scenario.
I threw together this simple profiler to check out the results of a worst case scenario. (I had to increase Java's stack size using -Xss to run this)

```
public class Main {

	private static final int NUMBER_OF_ITERATIONS= 12;
	private static final int STARTING_N = 256;

	public static void main(String[] args) {
		int n = STARTING_N;
		for(int index = 0; index < NUMBER_OF_ITERATIONS; index++){
			int[] array = new int[n];
			for(int i = 0; i < array.length;){
				array[i] = ++i;
			}
			QuickShuffle.quickShuffle(array, new Random(3));
			long start = System.currentTimeMillis();
			QuickSort.quickSort(array, new Random(3));
			long sameSeedResults = System.currentTimeMillis() - start;
			if(!isSorted(array))
				throw new RuntimeException();
			QuickShuffle.quickShuffle(array, new Random(3));
			start = System.currentTimeMillis();
			QuickSort.quickSort(array, new Random(4));
			long differentSeedResults = System.currentTimeMillis() - start;
			System.out.println("Same seed      : size = " + n + "\ttime = " + sameSeedResults);
			System.out.println("Different seeds: size = " + n + "\ttime = " + differentSeedResults);
			n <<= 1;
		}			
	}

	public static boolean isSorted(int[] array){
		for(int index = 0; index < array.length - 1; index++){
			if(array[index] > array[index + 1])
				return false;
		}
		return true;
	}
}
```

The profiler starts at n = 256 and then increases n by a power of 2 for each following iteration. Each iteration, an array of size n is created and filled with numbers from 1 - n. The array is shuffled and sorted using the same seed then the array is shuffled and sorted using different seeds. Here is the output from that profiler.

```
Same seed      : size = 256	time = 1
Different seeds: size = 256	time = 1
Same seed      : size = 512	time = 0
Different seeds: size = 512	time = 1
Same seed      : size = 1024	time = 1
Different seeds: size = 1024	time = 1
Same seed      : size = 2048	time = 6
Different seeds: size = 2048	time = 1
Same seed      : size = 4096	time = 3
Different seeds: size = 4096	time = 0
Same seed      : size = 8192	time = 12
Different seeds: size = 8192	time = 1
Same seed      : size = 16384	time = 47
Different seeds: size = 16384	time = 2
Same seed      : size = 32768	time = 187
Different seeds: size = 32768	time = 3
Same seed      : size = 65536	time = 743
Different seeds: size = 65536	time = 8
Same seed      : size = 131072	time = 2960
Different seeds: size = 131072	time = 19
Same seed      : size = 262144	time = 11936
Different seeds: size = 262144	time = 34
Same seed      : size = 524288	time = 48304
Different seeds: size = 524288	time = 72
```

Holy cow. Lets look at that in a graph. (They are sorting the exact same array!)
![Graph showing exponential curve](/images/20150202030320.png)

The sorter using the same seed as the shuffler definitely looks exponential while the sorter using a different seed is hardly seen at all.

## Conclusion

There is a possiblity to hit the worst case using a randomized pivot point for your quicksort algorithm, but I 'forgot' to mention that the possiblity of you finding one of these arrays that match up do perfectly in the 'wild' is extremely rare. Using a random pivot is still probably the safest way to do it. Just don't forget that it is not a guarantee that you will always perform at O(n*log(n)).
