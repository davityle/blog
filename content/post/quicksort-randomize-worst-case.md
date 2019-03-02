+++
date = "2015-10-03T10:01:16-07:00"
title = "Exploring the Worst Case Complexity of Quicksort"
Categories = ["algorithms"]
Tags = ["algorithms", "quicksort", "java", "big-o"]
+++

The quicksort algorithm has the best case complexity of O(n log n) when each pivot in the sort divides the list into two equal uniform pieces [1]. The worst case occurs when the pivot always divides the list into one list of size 1 and one list of size N - 1. Such as when sorting an already sorted array and the pivot is always chosen as the last element in the list. The complexity of quicksort in this case is an unfortunate O(n<span style="position: relative; bottom: 1ex; font-size: 80%;">2</span>). For this reason, it is recommended to use a random pivot point. This gives an average case of O(n log n) [2]. However, this does not remove the worst case scenario.

For example, consider this array:

{{< highlight javascript>}}
array = {44,88,7,2,1,999,1040,23,123,89,2009}
{{< /highlight >}}

with these given “random” pivots: 

{{< highlight javascript>}}
4,3,2,7,4,7,9,8,8,9
{{< /highlight >}}
You get this result:

<!--more-->

{{< highlight javascript>}}
pivot: 4, array[4] == 1
{{< /highlight >}}
the pivot index is 4, the value at index 4 is 1. Notice that 1 is the lowest number in the array. After the first iteration you end up with this array
{{< highlight javascript>}}
{1,88,7,2,44,999,1040,23,123,89,2009}
{{< /highlight >}}

Each successive pivot number is the next lowest number in the array. 

{{< highlight javascript>}}
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
{{< /highlight >}}

The above sort took n iterations and each iteration handled every number that wasn’t already sorted. This results in n operations over n iterations, a complexity of O(n<span style="position: relative; bottom: 1ex; font-size: 80%;">2</span>).

#### Experiment

For fun — yet no profit — I slapped together a quick proof of concept in Java that, given an array and a seed, shuffles the array such that when the array is sorted using quicksort and a random pivot with the same seed, quicksort hits the worst case scenario [3].

I profiled quicksort when run against a shuffled array — both with the shuffle seed and with another control seed [4] [5]. The results were actually quite fun.

{{< highlight javascript>}}
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
{{< /highlight>}}

I only tested 12 iterations on my old laptop after the 12th iteration took almost a minute. 

![Graph showing exponential curve](/images/20150202030320.png)

You can clearly see in the results that the average case is exhibited when using the control seed and the worst case complexity is exhibited when using the same seed as the shuffle.

##### References

[1] https://en.wikipedia.org/wiki/Quicksort#Analysis_of_randomized_quicksort#Best-case_analysis

[2] https://en.wikipedia.org/wiki/Quicksort#Analysis_of_randomized_quicksort#Using_percentiles

[3] Shuffle Implementation

{{< highlight java>}}
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
{{< /highlight>}}

[4] Quick Sort Implementation

{{< highlight java>}}
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
{{< /highlight>}}


[5] Profile Implementation

{{< highlight java>}}
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
{{< /highlight>}}