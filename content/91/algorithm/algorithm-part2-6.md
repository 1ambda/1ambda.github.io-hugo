+++
date = "2016-06-25T13:01:22+09:00"
prev = "../algorithm-part2-5"
title = "Algorithm: Data Compression, Huffman, LZW"
toc = true
weight = 46
aliases = [
    "/algorithm-data-compression"
]
+++

### Data Compression

주된 이유는 전송 시간과 저장 공간을 절약하기 위해서다. 무어의 법칙이 말해주듯이 제품의 성능은 점점 좋아지는데, 그럼에도 불구하고 사람들이 만들어 내는 데이터의 양은 더 급격히 증가한다. 그래서 압축이 필요하다.

이번시간에 배울 기법은 3 가지다.

- Run-length
- Huffman
- LZW

*data compression* 응용은

- *generic file compression*

GZIP 같은 파일 압축이나, PKZIP 같은 아키이빙. 그리고 파일시스템에서도 압축을 할 수 있다.

- *multimedia*, *communication*, *database*

*GIF*, *MP3*, *V.42 bis model*(?) 등 다양한 곳에 압축을 활용한다고 한다.

<br/>

![](http://www.programering.com/images/remote/ZnJvbT1pdGV5ZSZ1cmw9Y21idzVTT2paRE5qTnpZMEVXTjNNVEw0Z1RONDBDTjFFMk10RWpad0VUTGtWRE41SWpNMGt6THlnVE0xOHlNNUFETXZRbmJsMUdhakZHZDBGMkxrRjJic0JYZHYwMmJqNVNaNVZHZHA1aU1zUjJMdm9EYzBSSGE.jpg)
<p align="center">(http://www.programering.com)</p>

강의에서 사용하는 용어를 좀 정리하고 가자. 바이너리 `B` 에 대해 *compression, 압축* `C(B)` 를 하고 *expansion, 복원(?)* 를 해서 `B` 를 얻는다. 중요한점은 *loseless* 여야 한다는 것.

*compression ratio* 는 `C(B) / B` 로 정의한다. *natural language* 의 경우 `50~75%` 이상의 압축이 가능하다고 한다.

#### Binary Stream

구현에 사용할 바이너리 스트림 API 를 보자. 코드는 [BinaryStdIn.java](http://introcs.cs.princeton.edu/java/stdlib/BinaryStdIn.java.html) [BinaryStdOut.java](http://introcs.cs.princeton.edu/java/stdlib/BinaryStdOut.java.html) 에서 구할 수 있다.

```java
public class BinaryStdIn {
    boolean readBoolean() {} //read 1 bit and return as a boolean
    char readChar() {} //read 8 bits and return as a char
    char readChar(int r) {} //read r bitsand return as a char
    [similar methods for byte (8 bits); short (16 bits); int (32 bits); long and double (64 bits)]
    boolean isEmpty() {} //is the bitstream empty?
    void close() {} //close the bitstream
}

public class BinaryStdOut {
    void write(boolean b) {} //write the specified bit
    void write(char c) {} //write the specified 8-bit char
    void write(char c, int r) {} //write the r least significant bits of the specified char
    [similar methods for byte (8 bits); short (16 bits); int (32 bits); long and double (64 bits)]
    void close() {} //close the bitstream
}
```

찾아보니까 자바에서 기본적으로 제공되는건 `ByteArrayStream` 이 있더라.

#### Data Representation

`12/31/1999` 를 표시하는 법을 생각해 보자.

(1) `char` 면 *8 bit*  10개니까 *80 bit*  
(2) `int`  면 *32 bit*  3개니까 *96 bit*  
(3) `bit` 단위를 지정할 수 있다면 *4 + 5 + 12 + 3(align) = 24 bit* 

```java
month = 12
day = 31
year = 199

BinaryStdOut.write(month, 4)
BinaryStdOut.write(day, 5)
BinaryStdOut.write(year, 12)
```

이는 데이터 표현만 잘 해도 알고리즘 없이 상당부분 압축할 수 있다는 소리다. 일레로 1990년도에 만들어졌던 유전자 데이터베이스는 아스키로 `AGCT` 를 표현했다고 한다. 4종류의 문자밖에 없으니까 *2 bit* 로 표현 가능함에도 불구하고

### Universal Data Compression

> No algorithm can compress every bitstring

*contraction* 을 이용해 증명하면

- 모든 비트스트림을 압축할 수 있는 알고리즘 `U` 가 있다 하자.
- 주어진 비트스트링 `b0` 을 압축해서 더 작은 사이즈의 `b1` 을 얻고
- 이 과정을 반복하면 사이즈가 `0` 이 될때까지 압축이 가능하다. 이건 말이 안된다. 
- 따라서 모든 비트스트림을 압축할 수 있는 알고리즘 `U` 는 없다.

### Run-Length Encoding

`0000000000000001111111000000011111111111` 의 비트가 있을때 *n-bit* 로 `0` 또는 `1` 의 *runs* (긴 나열을 말하는 듯함) 을 표시한다. 

예를 들어 위의 데이터를 4비트 카운트를 이용해 표시하면

```
bin  1111 0111 0111 1011
dec    15    7    7   11
```

만약에 *run* 의 길이가 지정된 *n-bit* 로 표시할 수 있는 수보다 크면 `0` 부터 다시 세면 된다. *JPEG*, *ITU-T T4 Group 3* 등에 응용한다고 함.

구현은

```java
// http://algs4.cs.princeton.edu/55compression/RunLength.java.html
public class RunLength {

  private final static int R = 256; // max run-length count
  private final static int lgR = 8; // # of bits per count
  
  public static void compress() {
    int run = 0;
    boolean old = false;
    
    while (!BinaryStdIn.isEmpty()) {
      boolean current = BinaryStdIn.readBoolean();
      
      // alternate bit
      if (current != old) {
        BinaryStdOut.write(run, lgR);
        run = 1;
        old = !old;
      }
      // same bit
      else {
        // max count
        if (run == R - 1) {
          BinaryStdOut.write(run, lgR);
          // print dummy alternate bit whose length is 0
          run = 0;
          BinaryStdOut.write(run, lgR); 
        }
        
        run++;
      }
    }
    
    BinaryStdOut.write(run, lgR);
    BinaryStdOut.close();
  }
  
  public static void expand() {
    boolean bit = false;
    
    while (!BinaryStdIn.isEmpty()) {
      int run = BinaryStdIn.readInt(lgR); // read lgR bit
      
      for (int i = 0; i < run; i ++)
        BinaryStdOut.write(bit);
      
      bit = !bit;
    }
    
    BinaryStdOut.close();
  }
}
```

~~테스틀 어찌해야하는가~~

<br/>

![](http://help.adobe.com/en_US/Director/11.0/images/vector_bitmap_image.png)
<p align="center">(http://help.adobe.com)</p>

*bitmap* 을 압축하는데 *run-length* 를 사용하면 효과적이다. 글자에서 대부분의 비트가 `0` (흰색) 이기 때문이다.

흑백 그림을 예로 들어보자. 인치당 `300` 픽셀이고, 사이즈가 `8.5 x 11` 인치라 했을때, 한 이미지를 표시하기 위해 필요한 비트는 `300 * 8.5 * 300 * 11` = `8.415` 백만 비트가 필요하다.

### Huffman Encoding

*fixed-length code* 말고 모스코드같은 *variable-length code* 를 생각해 보자. 

모스코드에서 `* * * - - - * * *` 는 여러 방법으로 해석될 수 있다. `SOS`, `V7`, `IAMIE`, `EEWNI` 모두 가능하다. 모호한것이다. 모스부호에서는 이 문제를 해결하기 위해 글자마다 갭을 두어, 올바르게 해석될 수 있도록 한다. 

인코딩에서 *ambiguity* 를 해결하려면 어떤 *codeword* 도 다른 *code word* 의 *prefix* 가 되지 않도록 해야 한다. 

- Fixed-length code
- Append special stop char to each codeword (e.g 모스)
- Prefix-free code

이 중에서 *prefix-free* 인 코드를 만드는법을 살펴보자. *binary trie* 를 만들어 *leaf* 에 문자를 놓고, 그 문자까지 도달하는 경로가 인코딩 값이다. 

![](http://www.programering.com/images/remote/ZnJvbT1pdGV5ZSZ1cmw9Y21idzVpTjJRVE9tTnpZemdETjVVVEw0UTJZaTF5WXdVek10WWpNNUlXTDJRek5oZFRPaWR6THlrVE0xOHlNNUFETXZRbmJsMUdhakZHZDBGMkxrRjJic0JYZHYwMmJqNVNaNVZHZHA1aU1zUjJMdm9EYzBSSGE.jpg)
<p align="center">(http://www.programering.com)</p>

*compression* 을 위해 심볼테이블을 만들거나, 아니면 *leaf* 부터 따라 올라가고, 그 *path* 를 뒤집어 출력할 수 있다.

*expansion* 은 루트부터 시작해서 *path* 를 따라 내려가다가 *leaf* 에서 만나는 문자를 출력하면 된다.

한가지 생각해 볼 것은, 빈도가 많은 문자를 짧은 인코딩 값(*path*) 를 가지도록 해야 압축률이 높아진다는 것이다. 이를 위해 각 문자의 *frequency* 를 이용해야 한다.

허프만 코드 API를 보면

```java
public class Huffman {

	// support extended ASCII
	private static final int R = 256;
	
	private static class Node implements Comparable<Node> {

		private char ch;
		private int  freq;
		private final Node left, right;
	
		public Node(char ch, int freq, Node left, Node right) {
			this.ch = ch;
			this.freq = freq;
			this.left = left;
			this.right = right;
		}
	
		public boolean isLeaf() {
			return left == null && right == null;
		}

		@Override
		public int compareTo(Node that) {
			return this.freq - that.freq;
		}
	}
	
	public static void compress()
	public static void expand()
	private static Node buildTrie(int[] freq)
	private static void writeTrie(Node x)
	private static void buildCode(String[] st, Node x, String s)
	private static Node readTrie()
}
```

이제 *expansion* 을 구현하자. 스트림의 가장 첫 부분에, 몇 개의 문자인지를 *int* 로 표시하는 규약을 정하면

```java
public void expand() {
  Node root = readTrie(); // read in encoding trie
  int N = BinaryStdIn.readInt(); // read in # of chars
  
  for (int i = 0; i < N; i++) {
    Node x = root;
    while (!x.isLeaf()) {
      if (!BinaryStdIn.readBoolean())
        x = x.left;  // 0
      else
        x = x.right; // 1
    }
    
    BinaryStdOut.write(x.ch, 8); // print char
  }
  
  BinaryStdOut.close();
}
```

*running time* 은 `N` 에 비례한다. 

*expansion* 하는 쪽에서도 *trie* 를 가지고 있어야한다. *trie* 를 전송하기 위해 `writeTrie` 함수를 만들어 보자. *trie* 를 *preorder* 로 순회하면서 *leaf* 의 경우 `1` 과 문자 값을, *internal node* 의 경우 `0` 을 출력한다.

```scala
private static void writeTrie(Node x) {
  if (x.isLeaf()) {
    BinaryStdOut.write(true); // leaf
    BinaryStdOut.write(x.ch, 8);
    return;
  }
  
  BinaryStdOut.write(false);
  writeTrie(x.left);
  writeTrie(x.right);
}
```

이러면 비트스트림으로 온 *trie* 를 해석하는 함수 `readTrie` 를 만들자. 마찬가지로 *pre-order* 로 읽는다.

```java
private static Node readTrie() {
  // leaf
  if (BinaryStdIn.readBoolean()) {
    char c = BinaryStdIn.readChar(8);
    return new Node(c, 0, null, null);
  }
  
  Node l = readTrie(); 
  Node r = readTrie();
  return new Node('\0', 0, l, r);
}
```

#### Shannon-Fano algorithm

어떻게 가장 최적(압축률이 높은) *prefix-free code* 를 만들까? *Shannon-Fano* 알고리즘을 이용하면

- *symbol* `S` 를 `freq` 으 합이 최대한 같은 두 집단으로 나눈다 `S0, S1`
- `S0` 은 `0` 부터 시작하고, `S1` 은 `1` 부터 시작하도록 *codeword* 를 만든다
- 위 두 단계를 반복한다.

근데 이 알고리즘은 보면 알겠지만, 최적이 아니다. 이는 `freq` 값이 `{5, 1}`, `{2, 1, 2, 1}` 처럼 구성될 수 있기 때문이다.

#### Huffman algorithm

허프만 알고리즘은 최적의 *prefix-free code* 를 만들기 위해 

- 각 문자를 이용해 *single node trie* 를 만든다.
- `freq` 값이 가장 작은 두개를 골라 합치고, *internal node* 에 값을 누적
- 반복한다

![](http://lcm.csa.iisc.ernet.in/dsa/img161.gif)
<p align="center">(http://lcm.csa.iisc.ernet.in/dsa)</p>

이 과정에서 가장 낮은 *frequency* 를 갖는 문자가 아래로 가는 것을 보장하기 때문에 최적의 *prefix-free code* 를 찾는다 말할 수 있다. (좀 더 자세한 증명은 책을 보라고 함)

구현은

```java
// construct huffman encoding trie 
private static Node buildTree(int[] freq) {
	PriorityQueue<Node> pq = new PriorityQueue<Node>();
		
	for (char c = 0; c < R; c++) 
		if (freq[c] > 0)
			pq.add(new Node(c, freq[c], null, null));
		
	// if only one char
	if (pq.size() == 1) {
		if (freq['\0'] == 0) pq.add(new Node('\0', 0, null, null));
		else                 pq.add(new Node('\1', 0, null, null));
	}
		
	// merge two tries
	while (pq.size() > 1) {
		Node l = pq.remove();
		Node r = pq.remove();
		Node p = new Node('\0', l.freq + r.freq, l, r); // parent
		pq.add(p);
	}
	
	return pq.remove();
}
```

#### Compression

구현은 

```java
public static void compress() {
	String s = BinaryStdIn.readString(); // input
	char[] input = s.toCharArray();
	
	// tabulate freq counts
	int[] freq = new int[R];
	for (int i = 0; i < R; i++)
		freq[input[i]]++;
	
	// build huffman trie
	Node root = buildTrie(freq);
	
	// build syombol table
	String[] st = new String[R];
	buildCode(st, root, "");
	
	// print trie for decoder
	writeTrie(root);
	
	// print N (# of input)
	BinaryStdOut.write(input.length);
	
	// encode
	for (int i = 0; i < input.length; i++) {
		String code = st[input[i]];
		
		// traverse huffman trie
		for (int j = 0; j < code.length(); i++) {
			if (code.charAt(j) == '0')
				BinaryStdOut.write(false);
			else if (code.charAt(j) == '1')
				BinaryStdOut.write(true);
			else throw new IllegalStateException("Illegal State");
		}
	}
	
	BinaryStdOut.close();
}

private static void buildCode(String[] st, Node x, String s) {
	if (!x.isLeaf()) {
		buildCode(st, x.left, s + '0');
		buildCode(st, x.right, s + '1');
	} else {
		st[x.ch] = s; 
	}
}
```

전체 코드는 [Huffman.java](ko.thetimenow.com/pst/pacific_standard_time) 로

#### Huffman Summary

정리하면 

- tabulate char frequencies and build trie
- encode file by traversing trie or lookup table

러닝타임은 바이너리 힙을 이용할 경우(우선순위 큐) `N + RlogR` 이다. 여기서 `R` 은 알파벳 사이즈. `N` 은 입력 문자의 수다. 

즉 `N` 은 입력 문자를 인코딩 하는데, `R logR` 은 `R` 개의 문자에 대해 `freq` 값을 이용해 *trie* 를 만드는데 걸리는 시간

### LZW Compression	

*Lempel-Ziv-Welch* 의 약자다. 세 분이 만드신듯

알고리즘을 보기 전에 데이터 압축 모델에 대해 좀 생각해보자.

(1) 빠르고, 범용적이지만 최적은 아닌 **static model** (e.g ASCII)
(2) 모델을 매번 생성하고, 전송해야하지만 최적인 **Dynamic model** (e.g Huffman)

이 둘을 섞은 *adaptive model* 도 있다. 즉 매 텍스트마다 모델을 업그레이드 해 나가는 것이다. ~~머신러닝?!~~

> Progressively learn and upate model as you read text

- More accurate modeling produces better compression
- Decoding must start from beginning

*LZW compression* 이 그 예다. *LZW* 압축 알고리즘은 모델을 읽으면서 만들기 때문에, 전송할 필요가 없다.

텍스트를 읽다가 *codeword table* 에 이미 존재하면, 문자를 더 읽어 테이블에 없을 경우에만 추가한다. 그림으로 보면

![](http://www.programering.com/images/remote/ZnJvbT1pdGV5ZSZ1cmw9Y21idzVDWmlSR00yUW1OeVF6WWpaVEw0UTJNNDBTTjFVMk10a3pZM01XTHlFV05obGpZNU0yTHhBak0xOHlNNUFETXZRbmJsMUdhakZHZDBGMkxrRjJic0JYZHYwMmJqNVNaNVZHZHA1aU1zUjJMdm9EYzBSSGE.jpg)

- Create ST associating `W`-bit codewords with string keys
- Initialize ST with codewords for single-char keys
- Find longest string `s` in ST that is a prefix of unscanned part of input
- Write the `W`-bit codeword associated with `s`
- Add `s + c` to ST where `c` is next char in the input

*LZW compression* 에선 *longest string matching* 이 필요하므로 *trie* 를 이용할 수 있다.

![](http://www.programering.com/images/remote/ZnJvbT1pdGV5ZSZ1cmw9Y21idzVDWmtsek5pRldPbVJtTmpSVEw1RVRZNTBDT3pFek10UVRZMElXTG1GVFk1TVdPbVZ6THpBak0xOHlNNUFETXZRbmJsMUdhakZHZDBGMkxrRjJic0JYZHYwMmJqNVNaNVZHZHA1aU1zUjJMdm9EYzBSSGE.jpg)
<p align="center">(http://www.programering.com)</p>

#### LZW Implementation

자세한 코드는 [LZW.java](http://algs4.cs.princeton.edu/55compression/LZW.java.html) 여기로. 그리고 메모리 사용량을 줄이기 위해 *ternary search trie* 를 이용한다. [TST.java](http://algs4.cs.princeton.edu/55compression/TST.java.html)

```java
// ref: http://algs4.cs.princeton.edu/55compression/LZW.java.html
private static final int R = 256;  // # of input chars
private static final int L = 4096; // # of codewords = 2^W
private static final int W = 12;   // codeword width

public static void compress() {
  String input = BinaryStdIn.readString();
  
  // codewords for single chars
  TST<Integer> tst = new TST<Integer>();
  for(int i = 0; i < R; i++)
    sts.put("" + (char) i, i);
  
  int code = R + 1;
  
  while (input.length() > 0) {
    String s = tst.longestPrefixOf(input);
    BinaryStdOut.write(s, W);
    
    int t = s.length();
    if (t < input.length() && code < L)
      st.put(input.substring(0, t + 1), code++)
    
    input = input.substring(t);
  }
  
  BinaryStdOut.write(R, W); // write "stop" codeword
  BinaryStdOut.close();
}
```

*expansion* 은 테이블에서 *codeword* 를 읽어가면서 테이블을 만들면 된다. 심볼이 아니라 값으로 검색하므로 `2^W` 길이의 *array* 만 있으면 된다.

*tricky case* 가 있는데

![](http://www.programering.com/images/remote/ZnJvbT1pdGV5ZSZ1cmw9Y21idzVpTTBNbVp3WVdaaloyWTRJVEwxY1RZaDF5TnlJMk10VUROaGRUTHdVVE4zWTJZM2t6TDFBak0xOHlNNUFETXZRbmJsMUdhakZHZDBGMkxrRjJic0JYZHYwMmJqNVNaNVZHZHA1aU1zUjJMdm9EYzBSSGE.jpg)

*compression* 은 같은데 *expansion* 이 좀 복잡하다. `41 42 81 83 80` 에서 `83` 을 해석할때, 테이블에 있어야 해석을 하는데 없으므로 막혀버린다. 

이 경우를 잘 보면 `83` 을 해석해야 하고, 현재 테이블에 `82` 까지의 심볼만 존재한다. 그리고 방금 전 까지 읽은건 `81` 이다. `83` 은 `ABA` 인데, 이건 `81` 의 심볼을 `val` 이라 하면 `val + val.charAt(0)` 과 동일하다.

구현은

```java
public static void expand() {
	String[] st = new String[L];
	int i; // next available codeword value
	
	for (i = 0; i < R; i++) 
		st[i] = "" + (char) i;
	
	st[i++] = ""; // termination codeword
	
	int codeword = BinaryStdIn.readInt(W);
	if (codeword == R) return; // if empty message
	String val = st[codeword];
	
	while(true) {
		BinaryStdOut.write(val);
		
		codeword = BinaryStdIn.readInt(W);
		if (codeword == R) break;
		
		String s = st[codeword];
		
		// tricky case
		if (codeword == i) s = val + val.charAt(0);
		// add string into table
		if (i < L) st[i++] = val + s.charAt(0);
		
		val = s;
	}
	
	BinaryStdOut.close();
}
```

해석할때 `val` 은 지난단계의 해석, `s` 는 현재 읽은 값의 해석이라 생각하면 쉽다.

#### Lossless Data Compression

*LZW* 같은 경우 특허가 있었는데, 2003년에 만료되었다고 한다. 이 알고리즘의 많은 변형이 있는데, *LZ77* 은 특허가 없어서 오픈소스에 많이 쓰인다고 함. *deflate  zlib* 는 *LZ77* 과 *Huffman* 을 섞여 쓰는 대표적인 압축 알고리즘

*bit / char* 를 기준으로 보면 아스키는 `7`, 허프만은 `4.7` 정도의 성능을 보여준다. 1995년에는 *Burrows-Wheeler* 알고리즘이 발명되었는데 `2.29` 정도까지 압축한다. 1999년에는 *RK* 알고리즘이 `1.89` 까지 압축에 성공함.

### Summary

- **Huffman:** represent *fixed-length symbols* with *variable-length codes*
- **LZW:** represent *variable-length symbols* with *fixed-length codes*

*lossy compression* 의 경우 *FFT*, 프랙탈 등 수학적 도구를 이용해 만드 알고리즘들이 많다.

그리고 압축의 이론적 한계는 *shanon entropy* 에 의해

![](http://crackingthenutshell.com/wp-content/uploads/file/shannons-formula-small.jpg)

### Reference

(1) *Algorithms: Part 2* by **Robert Sedgewick**  
(2) [http://introcs.cs.princeton.edu](http://introcs.cs.princeton.edu/java/73dfa/)  
(3) [The NSA and Encryption](http://www.lawfareblog.com/2013/09/the-nsa-and-encryption/)  
(4) [Data Compression Lecture Note](http://www.programering.com/a/MTO4YzNwATE.html)  
(5) [Huffman Algorithm](http://lcm.csa.iisc.ernet.in/dsa/node88.html)  
