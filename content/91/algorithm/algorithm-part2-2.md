+++
date = "2016-06-25T13:01:17+09:00"
next = "../algorithm-part2-3"
prev = "../algorithm-part2-1"
title = "Algorithm: Radix Sort, Suffix Sort"
toc = true
weight = 42
aliases = [
    "/radix-sort-suffix-sort"
]
+++

### Strings in Java

문자열은 *Character (문자)* 의 나열이다. C 에서 하나의 캐릭터는 *8-bit* 인데, 자바의 경우에는 *16-bit unsigned integer* 로 표시한다. 

스트링의 길이를 얻기 위해 `length`, 인덱싱 하기 위해 `charAt`, 서브스트링을 얻기 위해 `substring` 의 메소드를 지원한다.

```java
public final class String implements Comparable<String> {

  private char values;
  private int offset; // index of first char in array
  private int length;
  private int hash; // cache of hashCode()
  
  ...
  
  public char charAt(int i) {
    return value[i + offset];
  }
  
  ...
}
```

자바에서 문자열은 *immutable* 이다. 더 정확히는 *immutable* `char []` *array* 라 보면 된다. 길이 정보를 가지고 있고 배열이기 때문에 `length`, `charAt`, `substring` 등의 연산은 `O(1)` 임을 보장한다.

`concat` 의 경우에는 새로운 문자열을 만들기 때문에 `O(N)` 이다. 메모리는 길이 `N` 의 문자열에 대해 `40 + 2N` 을 필요로 한다. 메모리를 아껴야 한다면 *byte, char* 을 이용할 수 있겠지만 여러 편리한 스트링의 메소드를 사용하지 못한다.

### StringBuilder, StringBuffer

`StringBuilder` 는 *mutable* 이다. `char []` 배열을 *resizing* 하기 때문에 

- `substirng` 의 경우 `O(N)` 이며 (새 스트링을 만든다)
- `concat` 은 `O(1*)` 이다. (`*` 는 amortized)

`length, charAt` 은 마찬가지로 `O(1)` 이다. 참고로 `StringBuffer` 는 `StringBuilder` 와 비슷하지만 *thread safe* 하고, 느리다.

그러면 *reverse* 를 구현 할 때 `String` 과 `StringBuilder` 중 어떤 것이 더 나을까?

```java
// 1. use String
public static String reverse(String s) {
  String rev = "";
  for (int i = s.length() - 1; i >= 0; i--)
    rev += s.charAt(i);
    
  return rev;
}  

// 2. use StringBuilder
public static String reverse(String s) {
  StringBuilder rev = new StringBuilder();
  for (int i = s.length() - 1; i >= 0; i--)
    rev.append(s.charAt(i));
    
  return rev.toString();
}  
```

`String` 을 이용한 버전은 `O(n^2)` 이고, `StringBuilder` 를 이용한 버전은 `O(n)` 이다. 이는 `+=` 와 `append` 의 차이 때문이다.

*suffixes* 문제도 생각해 보자.

```java
// input string
a a c a a g t t a c a a g c

// output
c     // suffixes 14
g c   // suffixes 13
a g c // suffixes 12
...
...
a a c a a g t t a c a a g c // suffixes 0
```

`String` 과 `StringBuilder` 의 구현을 생각해 보면,

```java
// 1. use String 
public static String[] suffixes(String s) {
  int N = s.length();
  String[] suffixes = new String[N];
  
  for (int i = 0; i < N; i ++)
    suffixes[i] = s.substring(i, N);
    
  return suffixes;
}

// 2. use StringBuilder 
public static String[] suffixes(String s) {
  int N = s.length;
  stringBuilder sb = new StringBuilder(s);
  String suffixes = new String
  
  for (int i = 0; i < N; i++)
    suffixes[i] = s.substring(i, N);
    
  return suffixes;
}
```

당연히 `substring` 은 `String` 이 메모리 사용량이 훨씬 더 적을꺼라 생각했는데 Java7 Update6 부터 좀 달라졌다고 한다.

*Java 7 Update 6* 부터는 이전처럼 `String` 의 `char []` 가 공유되지 않는단다. 따라서 `String.substring` 은 더이상 *constance space, time* 이 아니라 *linear space, time* 의 비용이 든다. 자세한 내용은 [Changes to String Java 1.7.0-06](http://java-performance.info/changes-to-string-java-1-7-0_06/)로 

따라서 알고리즘 `1` 은 *linear time, space* `2` 는 *quadratic time, space* 의 알고리즘이다.

#### Longest common prefix

```java
public static int lcp(String s, String t) {
  int n = Math.min(s.length(), t.length());
  
  for (int i = 0; i < n; i++)
    if (s.charAt(i) != t.charAt(i))
      return i;
      
  return n;
}
```

러닝타임은 `s, t` 중 더 긴 문자열의 길이에 비례한다. 일반적으로는 *sublinear time* 이다. 따라서 `compareTo` 메소드를 *sublinear time* 으로 구현할 수 있다.

#### Radix

알파벳을 다양한 형태로 표현할 수 있는데, *binary* 의 경우엔 `01` 이 될 것이다. 이때의 *radix* 는 2 다. *DNS* 는 `ACTG` 로 표현할 수 있으므로 `R = 4` 다.

### Key-Indexed Counting

정렬 알고리즘의 성능을 정리해 보면,

(1) **Insertion Sort**

- **guarantee:** `O(N^2 / 2)`
- **random:** `O(N^2 / 4)`
- **extra space:** `1`
- **stable:** `yes`

(2) **Merge Sort**

- **guarantee:** `O(N log N)`
- **random:** `O(N log N)`
- **extra space:** `N`
- **stable:** `yes`

(3) **Quick Sort**

- **guarantee:** `O(1.39 N log N)`
- **random:** `O(1.39 N log N)`
- **extra space:** `c log N`
- **stable:** `no`

(4) **Heap Sort**

- **guarantee:** `O(2 N log N)`
- **random:** `O(2 N log N)`
- **extra space:** `1 log N`
- **stable:** `no`

이런 *comparison based* 알고리즘은 *lower bound* 가 `N log N` 이다. 따라서 *key compare* 를 하지 않는다면 더 나은 성능을 낼 수 있다.

*key-indexed counting* 에서는 *key* 가 `0` 부터 `R - 1` 사이의 정수라 가정한다. 따라서 키를 배열의 인덱스로 사용할 수 있다.

따라서 다음처럼 활용할 수 있다.

- Sort String by first letter
- Sort class roster by section
- Sort phone number by area code
- Subroutine in a sorting algorithm

알고리즘을 보자. 

> **Goal:** Sort an array `a[]` of `N` integers between `0` and `R - 1`

(1) Count frequencies of each letter using key as index  
(2) Compute frequecy cumulates which specify destinations  
(3) Access cumulates using key as index to move items  
(4) Copy back into original array

```java
int N = a.length();
int[] count = new int[R + 1];

// step (1)
for (int i = 0; i < N; i++)
  count[a[i] + 1]++;
  
// step (2)
for (int r = 0; r < R; r++)
  count[r + 1] += count[r];

// step (3)
for (int i = 0; i < N; i++)
  aux[count[a[i]]++] = a[i];

// step (4)
for (int i = 0; i < N; i++)
  a[i] = aux[i];
```

- `~11N + 4R` *array access*
- `N + R` *extra space*

*key-indexed counting* 은 *linear time, stable sorting* 이다.

#### Stable

알고리즘이 *stable* 하다는 건 무슨 뜻일까? 

> A stable sort is one which preserves the original order of the input set while The unstable algorithm exhibits undefined behaviour when two elements are equal, it is perfectly possible that the order is sometimes preserved.

![](http://i.stack.imgur.com/hn6Rg.png)
<p align="center">(http://programmers.stackexchange.com/)</p>

### LSD Radix Sort

*least-significant-digit-first string(radix) sort*

아이디어는 간단하다. 우측부터 좌측으로 한 문자씩 *key-indexed couting* 을 하면 된다.

![](http://www.programering.com/images/remote/ZnJvbT1pdGV5ZSZ1cmw9Y21idzVpTjVFR1ozVWpNaVJHTzVZV0x4QXpZNDBpWjNJMk10WW1ZeEVUTHlZak5pVkdOMWN6TDJnRE14OHlNNUFETXZRbmJsMUdhakZHZDBGMkxrRjJic0JYZHYwMmJqNVNaNVZHZHA1aU1zUjJMdm9EYzBSSGE.jpg)
<p align="center">(www.programering.com)</p>

> Which of the following is the most efficient algorithm to sort 1 million 32-bit integers?

답은 *radix sort*

#### Correctness

> LSD sorts fixe-length strins in ascending order

가설에 의해 `i` 번째 pass 후에는 뒤 부터 `i` 개의 문자들이 정렬되어 있다.  이 때 비교하려는 `i+1` 번째의 두 문자가 다르다면, *key-indexed sort* 가 두 개의 문자열을 정렬한다. 이 때 *key-indexed sort* 는 *stable* 하므로 이전 까지의 정렬했던 순서를 보존한다.

#### Implementation

```java
	// W: fixed-length of strings
	public static void LSDsort(String[] a, int W) {
		int N = a.length;
		int R = 256;
		String[] aux = new String[N];
		
		// key indexed counting for each digit from right to left
		for (int d = W - 1; d >= 0; d--) {
			int[] count = new int[R + 1];
			
			// count frequencies
			for (int i = 0; i < N; i++) 
				count[a[i].charAt(d) + 1]++;
				
			for (int r = 0; r < R; r++) 	
				count[r + 1] += count[r];
			
			for (int i = 0; i < N; i++)
				aux[count[a[i].charAt(d)]++] = a[i];
		
			for (int i = 0; i < N; i++)
				a[i] = aux[i];
		}
	}
```

*LSD sort* 퍼포먼스는 `2WN`, 랜덤하게 `2WN`, 공간은 `N + R`, *stable* 하다. 참고로, `4byte Int` 에 대해 `1Byte` 씩 *LSD sort* 를 적용하면 `Array.sort` 보다 2~3배 더 빠르다고 한다. [코드는 여기로](http://algs4.cs.princeton.edu/51radix/LSD.java.html)

### MSD Radix Sort

*most significant-digit-first string sort*

- Partition array into `R` pieces according to first character
- Recursively sort all strings that start with each character

좌측 문자열 부터 시작하고, 현재 문자가 같은 문자열들 끼리 모아, 나머지 부분을 *sub-array* 취급해서 재귀적으로 정렬한다.

![](http://www.programering.com/images/remote/ZnJvbT1pdGV5ZSZ1cmw9Y21idzVDWjNNV001VW1ZaWhETTRjVExoTldONDBpTTJVMk10RVdaeEFUTHpVRFptZHpNaEYyTDVFVE14OHlNNUFETXZRbmJsMUdhakZHZDBGMkxrRjJic0JYZHYwMmJqNVNaNVZHZHA1aU1zUjJMdm9EYzBSSGE.jpg)
<p align="center">(www.programering.com)</p>

*LSD* 는 다루지 못하는 *variable-length string* 을 정렬할 수 있다.

#### Implementation


참고로 자바에서는 `\0` 이 없다. 그래서 문자열의 길이를 넘어서는 인덱스에 대해 `-1` 을 돌려주는 `charAt` 을 만들자. 추가적인 문자가 없다면, 정렬된 것으로 보고 끝내면 된다. 

```java
private static int charAt(String s, int d) {
  if (d < s.length()) return s.charAt(d);
  else return -1;
}

	private static void msdSort(String[] a, String[] aux, int l, int h, int d) {
		
		if (h <= l) return;
		
		int R = 256;
		int[] count = new int[R + 2];
		
		// count frequencies
		for (int i = l; i <= h; i++) {
			int c = charAt(a[i], d);
			count[c + 2]++;
		}
		
		// accumulate
		for (int r = 0; r < R + 1; r++)
			count[r + 1] += count[r];
		
		// sort
		for (int i = l; i <= h; i++) {
			int c = charAt(a[i], d);
			aux[count[c + 1]++] = a[i];
		}
		
		// copy
		for (int i = l; i <= h; i++)
			a[i] = aux[i - l];
		
		// solve sub-arrays
		for (int r = 0; r < R; r++)
			msdSort(a, aux, l + count[r], l + count[r + 1] - 1, d + 1);
	}
	
	public static void MSDsort(String[] a) {
		String[] aux = new String[a.length];
		msdSort(a, aux, 0, a.length - 1, 0);
	}
```

그런데 이 구현은 몇 가지 문제가 있다.

(1) 매 재귀마다 `count` 배열을 만들고, 그 크기는 `R` 에 비례하기 때문에 `~11R + N` 의 성능을 갖는 *key-indexed counting* 연산이 *small subarray* 가 많아지면서 급격히 느려진다. 

(2) 조그마한 *sub-array* 에 대해 많은 수의 재귀가 호출된다.

이 문제를 해결하기 위해 적은 수의 *small array* 에 대해 *insertion sort* 를 사용하자. 

```java
	// substring comparison is much faster than charAt comparison
	private static boolean less(String v, String w, int d) {
		return v.substring(d).compareTo(w.substring(d)) < 0;
	}
	
	private static void isort(String[] a, String[] aux, int l, int h, int d) {
		// insertion sort
		for (int i = l; i <= h; i++)
			for (int j = i; j > l && less(a[j], a[j - 1], d); j--) {
				// swap a[j - 1], a[j]
				String temp = a[j - 1];
				a[j - 1] = a[j];
				a[j] = temp;
			}
			
	}
	
	private static void msdSort(String[] a, String[] aux, int l, int h, int d) {
		
		int CUTOFF = 15;
		if (h <= l + CUTOFF) {
			isort(a, aux, l, h, d);
			return;
		}
    ...
    ...
    ...
```

#### Performance

*MSD string sort* 는 필요한만큼 *character* 를 확인하기 때문에, 데이터에 따라 성능이 다르다. 그러나 대부분의 경우 *sublinear* 하고, 운이 나쁜 경우 *linear* 의 성능이 나온다. *duplicated key* 가 있는 경우에는 *nearly linear* 다.

재밌는 사실은 `compareTo` 를 이용하지만 *sublinear* 하게 성능이 나올 수도 있다는 점이다.

![](http://www.programering.com/images/remote/ZnJvbT1pdGV5ZSZ1cmw9Y21idzVpTWhkVFk0VXpZa2RqTmtaV0x3Z2paNTBpTTNZek10VWpNMFVUTGxSMlkxY1Raamx6TDRJVE14OHlNNUFETXZRbmJsMUdhakZHZDBGMkxrRjJic0JYZHYwMmJqNVNaNVZHZHA1aU1zUjJMdm9EYzBSSGE.jpg)
<p align="center">(www.programering.com)</p>

*MSD string sort* 는 매 재귀마다 새로운 `count` 배열을 만들기 때문에 `N + DR` 만큼의 메모리가 필요하다., (`D` 는 재귀 호출의 수)

*LSD* 에 비해서 가변길이 문자열을 정렬할 수 있고, *random* 데이터에 대해 `N log_R N` 의 성능을 보여준다. *LSD* 와 마찬가지로 *stable* 하다.

#### MSD vs quick

*MSD string sort* 는 *random access* 를 하기 때문에 *cache inefficient* 할 수 있고, *quicksort* 에 비해 *inner loop* 에 많은 명령어가 있다. 게다가 `count, aux` 등 추가적인 메모리가 필요하다.

반면 *quicksort* 는 *linear* 하지 않다. 그리고, 많은 수의 문자들을 다시 비교해야한다. 이 두가지를 합친 방법은 없을까?

### 3-way Radix Quicksort

~~무려 교수님이 만드신 알고리즘 1997년에 이 수업에서 만들었다고 함~~

기본적인 아이디어는

> Do 3-way partitioning on the `d` th character

- Less overhead than `R`-way partitioning in *MSD string sort*
- Does not re-examine characters equal to the partitioning char (but does re-examine characters not equal to the partitioning char)

![](http://www.programering.com/images/remote/ZnJvbT1pdGV5ZSZ1cmw9Y21idzVpTWhKVFpoTkRPMk1qWTRRV0xsTm1ZaTFDTjVFek10SVdNaVZUTHdZVFlsVldPbVoyTDBNVE14OHlNNUFETXZRbmJsMUdhakZHZDBGMkxrRjJic0JYZHYwMmJqNVNaNVZHZHA1aU1zUjJMdm9EYzBSSGE.jpg)
<p align="center">(www.programering.com)</p>

즉, 첫 문자열의 첫 번째 문자를 기준으로, 이것보다 큰 것, 작은 것, 같은 것 3개로 파티셔닝하면서 정렬하는 알고리즘이다. 

#### Implementation

구현은 퀵소트랑 상당히 유사하다.

```java
	// 3-Way Quicksort
	public static void Quicksort3way(String[] a) {
		qsort3way(a, 0, a.length - 1, 0);
	}
	
	// helper method for 3 way quicksort
	private static void swap(String[] a, int i, int j) {
		String temp = a[i];
		a[i] = a[j];
		a[j] = temp;
	}
	
	private static void qsort3way(String[] a, int l, int h, int d) {
		if (h <= l) return;
	
		int lt = l, gt = h;
		int v = charAt(a[l], d);
		int i = l + 1;
		
		// partition
		while (i <= gt) {
			int t = charAt(a[i], d);
			
			if      (t < v) swap(a, lt++, i++);
			else if (t > v) swap(a, i, gt--);
			else            i++;
		}
		// a[lo..lt-1] < v = a[lt..gt] < a[gt+1..hi]
		qsort3way(a, l, lt - 1, d);
		if (v >= 0) qsort3way(a, lt, gt, d + 1);
		qsort3way(a, gt + 1, h, d);
	}
```

*MSD string sort* 와 마찬가지로 `CUTOFF` 를 이용해서 작은 *sub-array* 를 *insetion sort* 로 정렬할 수 있다.

#### 3-way quicksort vs standard quicksor

일반적으로 *quicksort* 는 `compareTo` 를 기준으로 `~ 2N lnN` 의 성능을 보여주고, *long common prefixes* 가 있는 경우에 상당히 계산 비용이 비싸다. 이는 비교했던 문자열을 또 비교할 수 있기 때문이다.

그러나 *3-way string quicksort* 는 `charAt` 을 기준으로 `~ 2N lnN` 의 성능을 보인다. 그리고 같은 파티션에 대해 `d + 1` 로 재귀호출하기 때문에 같은 파티션 내에서는 비교했던 문자를 다시 계산하지 않는다. ~~갓 교수님~~

#### 3-way quicksort vs MSD sort

(1) **MSD string sort** 는

- 같은 `count` 값을 가진 문자열에 뜬금없이 접근하기 때문에 *cache-inefficient*
- 재귀마다 `count[]` 를 새로 만들어 너무 많은 메모리를 사용
- `count[], aux[]` 를 초기화하는데 너무 많은 오버헤드

(2) **3-way string quicksort** 는

- 더 짧은 *inner loop*
- `while` 을 이용해 순차적으로 접근하므로 *cache-friendly*
- *in-place*

![](http://www.programering.com/images/remote/ZnJvbT1pdGV5ZSZ1cmw9Y21idzVTTTJFV09tZFRPakoyTTVjVEwxWW1ZNDBDTWtoek10UVRZMUlXTDVVRE8wWUdNd1UyTHhRVE14OHlNNUFETXZRbmJsMUdhakZHZDBGMkxrRjJic0JYZHYwMmJqNVNaNVZHZHA1aU1zUjJMdm9EYzBSSGE.jpg)
<p align="center">(www.programering.com)</p>

### Suffix Arrays

*keyword-in-context search* 란

> Given a text of `N` chars, preprocess i to enable fast substring search (find all occurrences of query string ocntext)

쉽게 말해서 구글 검색창에 *world* 라고 치면 *hello world* 등이 자동으로 검색목록에 나오는것.

![](http://www.programering.com/images/remote/ZnJvbT1pdGV5ZSZ1cmw9Y21idzVDWmtkVFozRW1ZbFZtTmlGVEx6RVRPNDBTWTFZek10UVRNeFFXTGlKVFpqbERNekkyTHpRVE14OHlNNUFETXZRbmJsMUdhakZHZDBGMkxrRjJic0JYZHYwMmJqNVNaNVZHZHA1aU1zUjJMdm9EYzBSSGE.jpg)
<p align="center">(www.programering.com)</p>

*suffixes* 를 만든다음에 문자열 정렬을 해서 중복된 *suffix* 가 있는지 보면 된다. `String` 의 경우 `substring` 을 얻는데 `O(1)` 이이므로 *suffixes* 를 만드는데 `O(n)` 이다. 

그 후에 *binary search* 를 하면, 일치하는 문자열을 검색할 수 있다.

(1) **Preprocess:** *suffix sort* the text.  
(2) **Query:** *binary search* for query; scan until mismatch.  



### Longest repeated substring

> Gien a string of `N` characters, find the longest repeated substring.

유전자 지도 `a g t t a a t c g ~` 에서 일치하는 가장 긴 유전자 문자열을 찾아내는데 활용할 수 있다.

*data compression* 에도 활용 가능하다. 자주 반복되는 긴 패턴을 발견해 짧게 줄이면 용량을 상당히 줄일 수 있다.

악보를 이용해서 음악을 *visualization* 하는데도 활용할 수 있다.

#### Brute Force 

문자열의 길이가 `N`, 가장 긴 패턴의 길이가 `D` 라면 *worst case* `DN^2` 이다. 

#### Sorting solution

(1) form suffixes (`O(n)`)  
(2) sort suffixes (`O(n log n)`)  
(3) compute longest prefix between adjacent suffixes (`O(kn)`)  

```java
	// longrest common prefix
	public static String lcp(String v, String w) {
		
		int n = Math.min(v.length(), w.length());
		
		for (int i = 0; i < n; i++) {
			if (v.charAt(i) != w.charAt(i)) return v.substring(0, i);
		}
		
		return v.substring(0, n);
	}
	
    // longest repeated substring
	public static String lrs(String s) {
	
		int N = s.length();
		String[] suffixes = new String[N];
	
		// form suffixes
		for(int i = 0; i < N; i++)
			suffixes[i] = s.substring(i, N);

		// sort
		Arrays.sort(suffixes);
		
		// find longest repeated substring using lcp
		String lrs = "";
		
		for (int i = 0; i < N - 1; i++) {
			String x = lcp(suffixes[i], suffixes[i + 1]);
			
			if (x.length() > lrs.length()) lrs = x;
		}
		
		return lrs;
	}
```

*suffix soring* 에 *3-way string quicksort* 를 이용하면 어마어마하게 빠르다.

한 가지 문제는 *lrs* 의 길이가 길어지면 *suffix sort* 의 성능이 급격히 떨어진다. `D` 를 *lrs* 의 길이라 했을때 적어도 `1 + 2 + ... + D` 의  문자열 비교가 필요하다. (자신과 자신의 서브스트링과의 비교)

따라서 `D` 가 길면 길수록 성능이 나빠진다. 

> Quadratic (or worse) in `D` for *LRS*

성능이 떨어지는 입력 데이터로, 반복되는 인풋이 있다. `twinstwins` 를 예로 들면

```
ins
instwins
ns
nstwins
s
stwins
twins
twinstwins
wins
winstwins
```

그러면 더 빠른 알고리즘이 없을까? *Manber-Myers algorithm* 이란게 있는데, 요건 *linearithmic*

*suffix trees* 란 것도 있다. 이건 *linear*

#### Manber-Myers MSD Algorithm

(1) sort on first character using key-indexed counting sort  
(2) given array of suffixes sorted on first `2^(i-1)` characters (phase `i`)  

*worse-case* 퍼포먼스는 `N lgN`

![](http://www.programering.com/images/remote/ZnJvbT1pdGV5ZSZ1cmw9Y21idzVpWmtCVE9rZFRaMVFEWjVBVEwwY2paaTFTTjBjek10Z0RabU5XTGlSRE9qUkRNalIyTDNNak14OHlNNUFETXZRbmJsMUdhakZHZDBGMkxrRjJic0JYZHYwMmJqNVNaNVZHZHA1aU1zUjJMdm9EYzBSSGE.jpg)
<p align="center">(www.programering.com)</p>

*key-indexed counting* 을 이용해 먼저 하나의 문자를 정렬하고, 그 이후에는 `2, 4, 6, 8, ...,` 개씩 정렬해 나간다. 이 과정에서 `inverse[]` 를 이용한 *index sort* 란걸 하는데, 아이디어는 이렇다.

*suffixes* 에서 비교하려는 두 문자열의 뒷부분의 일부는 이미 이전 단계에 정렬 되었을 수 있다. (슬라이드의 빨간색 부분) 따라서 이미 정렬해 된 순서 `inverse[]` 를 이용해서 현재 비교하려는 두 문자열의 순서를 정할 수 있다. 

#### Summary

- *linear-time* 문자열 정렬을 할 수 있다.

왜냐하면 *key comparison* 이 아니라 *character comparison* 으로 해낼 수 있기 때문

- *sublinear-time* 정렬도 만들 수 있다.

모든 문자열을 비교할 필요가 없기 때문 (Input size is amount of data in keys, not number of keys.)

- *3-way string quicksort is asymptotically optimal*

`1.39 N lgN` 의 문자열 비교, *random data* 에 대해. 그러나 *suffix sort* 에 대해 `N lgN` 을 보장하려면(*worst case*) *Manber-Myer* 를 사용해야 한다.

- Long strings are rarely random in practice

### References

(1) *Algorithms: Part 2* by **Robert Sedgewick**  
(2) [What is a stable sorting algorithm?](http://programmers.stackexchange.com/questions/247440/what-does-it-mean-for-a-sorting-algorithm-to-be-stable)  
(3) [www.programering.com](http://www.programering.com/a/MTOyYjNwATM.html)
