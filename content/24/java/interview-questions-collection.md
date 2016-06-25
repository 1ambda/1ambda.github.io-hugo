+++
date = "2016-06-25T12:06:44+09:00"
prev = "/.."
title = "Interview Questions: Collection"
toc = true
weight = 11
aliases = [
    "/java-interview-questions-collection-framework"
]
+++

![](http://4.bp.blogspot.com/_b6abT-2H2yE/TSTixbyU8GI/AAAAAAAAAUU/LcqDidb_liw/s1600/screen-capture-1.png)

<br/>

### General

(1) **Explain Collections Hierarchy?**

![](http://2.bp.blogspot.com/-M0M8nv5s2lQ/U3BcbRQcRvI/AAAAAAAAAec/oBBmQCPDm9Y/s1600/Collection-Classes.tif)

![](http://4.bp.blogspot.com/-o9Jk4Z4Tohs/U3Be46CxGTI/AAAAAAAAAeo/Wq8-hhZ8dCA/s1600/Collection-Classes_Map.tif)

<p align="center">(http://www.java-redefined.com)</p>

크게 보면 *Collection* 과 *Map* 인터페이스로 구분되어 있습니다. 


- `Map` 은 *key-value pair* 컨테이너이기 때문에 단일 원소에 대한 컨테이너인 `Collection` 과 호환되지 않습니다.
- `Set` 은 중복된 원소를 허용하지 않습니다.
- `Set` 과 `Map` 에 정렬 기능이 필요하면 `SortedSet`,  `SortedMap` 인터페이스 구현체인 `TreeMap`, `TreeSet` 등을 이용할 수 있습니다.

(2) How do you remove an entry from a collection? and subsequently what is difference between `Collection.remove()` and `Iterator.remove()`, which one you will use, while removing elements during iteration?

아래에서 언급하겠지만 *fail-fast* 와 관련된 문제입니다. 만약 순회하고 있지 않다면 `Collection.remove()` 를 사용해도 상관 없지만

*iterator* 를 이용해서 순회하는 동안 컬렉션의 `remove()` 메소드를 이용하면 `ConcurrentModificationException` 예외가 발생합니다.  따라서 `Iterator.remove()` 를 이용해야 합니다. [SO 답변](http://stackoverflow.com/questions/14200489/collection-iterator-remove-vs-collection-remove)에서도 그 이유를 찾을 수 있습니다.

```java
// invalid
List<Integer> l = new ArrayList<Integer>(Arrays.asList(1, 2, 3, 4));
for (int el : l) {
  if (el < 3) {
      l.remove(el);
  }
}
    
// correct way
Iterator<Integer> it = l.iterator();
while (it.hasNext()) {
  int el = it.next();
  if (el < 3) {
      it.remove();
  }
}
```


<br/>

### List interface related

- `List` 는 중복된 원소를 허용하며 *ordered elements* 를 담는 컨테이너입니다. 때때로 *Sequence* 라 불리기도 합니다. 


(1) `Vector` vs `ArrayList` vs `LinkedList`

- `Vector` 의 모든 메소드는 *동기화 (synchronized)* 됩니다. `ArrayList` 는 *thread-unsafe* 합니다.
- `Vector` 는 *JDK* 첫 릴리즈부터 포함되어있던 레거시 클래스고, `ArrayList` 는 *JDK 1.2* 에서 컬렉션 프레임워크 도입과 함께 추가되었습니다.
- *default* 로 `Vector` 는 두배씩 사이즈가 커지는 반면, `ArrayList` 는 *50%* 씩 증가합니다.
- `LinkedList` 도 *thread-unsafe* 하기 때문에 대신 [ConcurrentLinkedQueue](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/ConcurrentLinkedQueue.html) 나 [LinkedBlockingDeque](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/LinkedBlockingDeque.html) 를 이용할 수 있습니다.

(2) What is different between `Iterator` and `ListIterator`

- `Iterator` 를 이용해 `Set` 등 컬렉션을 순회할 수 있지만 `ListIterator` 는 `List` 밖에 못 합니다
- `Iterator` 는 *forward-only* 지만, `ListIterator` 는 양방향 순회가 가능합니다
- `ListIterator` 는 *add*, *replace*, *getting index position* 등의 기능이 더 있습니다. 

참고로 *iterator* 를 이용해 리스트를 순회하는 방법은

```java
List<String> strList = new ArrayList<>();
Iterator<String> it = strList.iterator();

while(it.hasNext()){
  String obj = it.next();
  System.out.println(obj);
}
```


<br/>

### Set interface related

`Set` 은 *uniqueness of elements* 를 보장합니다. 따라서 중복된 원소를 허용하지 않습니다. 만약 *ordering* 이 있는 `Set` 을 사용하고 싶다면 구현체로 `TreeSet` 을 선택하면 됩니다.

(1) **How HashSet store elements?**

`HashSet` 은 *uniqueness* 를 보장하기 위해 내부적으로 `Map` 을 이용합니다. *key-value* 를 저장하나, 모든 *value* 를 같게끔 하죠.

```java
private transient HashMap<E, Object> map;
// This is added as value for each key
private static final Object PRESENT = new Object();

public boolean add(E e) {
  return map.put(e, PRESENT) == null;
}
```

(2) Can a null element added to a `TreeSet` or `HashSet`?

`HashMap`, `HashSet` 은 하나의 *null-key* 를 허용하지만, `TreeSet`, `TreeMap` 은 *null-key* 를 허용하지 않습니다. 

`TreeMap` 은 `NavigableMap` 의 구현이고, `TreeSet` 은 내부적으로 `NavigableMap` 을 사용합니다. 그런데 `NavigableMap` 이 *null-key* 를 허용하지 않기 때문에, `TreeSet`, `TreeMap` 도 그렇습니다.

<br/>

### Map interface related

`Map` 은 *key-value pair* 를 저장하기 위해 사용합니다. `Map` 인터페이스 구현체로 `HashMap`, `LinkedHashMap`, `HashTable`, `EnumMap`, `IdentityHashMap`, `Properties` 가 있습니다.

(1) Difference between `HashMap` and `HashTable`

- `HashTable` 은 *동기화 (synchronized)* 되지만, `HashMap` 은 그렇지 않습니다.
- `HashTable` 은 *null-key* 나 *null-value* 를 허용하지 않습니다.
- `HashMap` 의 *iterator* 는 **fail-fast** 인 반면 `HashTable` 의 *enumerator* 는 그렇지 않습니다.

참고로, *iterator* 는 *iteration* 동안 *caller* 가 `remove` 메소드를 이용해서 원소를 제거할 수 있지만, *enumerator* 를 이용할때는 원소를 추가하거나 제거할 수 없습니다. 이런 기능 차이 때문에 *enumerator* 가 기본적인 기능만 가지고 있고 더 빠릅니다. 또 다른 차이점은 *enumerator* 는 `Stack`, `Vector` 처럼 레거시 클래스에 대해 사용합니다.

(2) What are `IdentityHashMap` and `WeakHashMap`?

이부분은 [원문](http://howtodoinjava.com/2013/07/09/useful-java-collection-interview-questions/#identityHashMap_weakHashMap_differences)을 첨부합니다.

> **IdentityHashMap** is similar to HashMap except that **it uses reference equality when comparing elements**. IdentityHashMap class is not a widely used Map implementation. While this class implements the Map interface, it intentionally violates Map’s general contract, which mandates the use of the equals() method when comparing objects. IdentityHashMap is designed for use only in the rare cases wherein reference-equality semantics are required.

> **WeakHashMap** is an implementation of the Map interface that **stores only weak references to its keys**. Storing only weak references allows a key-value pair to be garbage collected when its key is no longer referenced outside of the WeakHashMap. This class is intended primarily for use with key objects whose equals methods test for object identity using the == operator. Once such a key is discarded it can never be recreated, so it is impossible to do a look-up of that key in a WeakHashMap at some later time and be surprised that its entry has been removed.

<br/>

### More Questions

(1) What do you understand by iterator **fail-fast** property?

**fail-fast iterator** 는 *iteration* 이 시작된 이후로 *collection* 이 변경되는걸 알아채는 순간 `ConcurrentModificationException` 을 던지면서 멈춥니다. 여기서 *변경* 이란 한 스레드가 컬렉션을 순회하는 동안, 컬렉션에 있는 원소의 삭제, 변경 혹은 추가가 일어나는 것을 말합니다.

*fail-fast* 는 *modification count* 란 것을 유지하고 있다가, *iteration thread* 가 *modification count* 의 변경을 알아채면 예외를 던지는 방식으로 구현됩니다.

(2) What is difference between **fail-fast** and **fail-safe**

**fail-safe iterator** 는 복사본에 대해 컬렉션 순회를 진행하기 때문에 원본에 변경이 일어나도 멈추지 않습니다. 일반적으로 `java.util.concurrent` 에 있는 클래스들의 (e.g `ConcurrentHashMap` 이나 `CopyOnWriteArrayList`) *iterator* 가 *fail-safe* 입니다.

(3) How to avoid `ConcurrentModificationException` while iterating a collection?

- 먼저 *fail-safe iterator* 를 사용할 수 있는지 확인합니다 *JDK 1.5* 이상을 사용한다면, `ConcurrentHashMap` 이나 `CopyOnWriteArrayList` 를 사용할 수 있습니다.

위 방법이 불가능하면 다음을 고려할 수 있으나, 퍼포먼스가 떨어질 수 있다는 점을 유의해야 합니다. 

- *list* 를 *array* 로 바꾸어, 순회합니다
- *list* 를 순회하는 동안 *synchronized block* 을 이용해 *lock* 을 겁니다.

(4) What is difference between Synchronized Collection and Concurrent Collection?

*Java 5* 와 함께 `ConcurrentHashMap`, `CopyOnWriteArrayList`, `BlockingQueue` 등의 *concurrent collection* 클래스들이 추가 되었습니다. 이 클래스들은 *synchronized collection* 보다 성능이 더 나은데, 이는 일부분에만 *lock* 을 걸기 때문입니다. 더 자세한 내용은 [여기로](http://javarevisited.blogspot.kr/2011/04/difference-between-concurrenthashmap.html)

(5) What is `Comparable` and `Comparator`?

자바에서 `TreeSet` 이나 `TreeMap` 처럼 *automatic sorting* 기능이 있는 모든 컬렉션은 `compare` 메소드를 사용합니다. 

이 때 *element class* 는 정렬을 위해 `Comparator` **또는** `Comparable` 인터페이스를 반드시 구현해야 합니다. *wrapper class* 인 `Integer`, `Double` 등이 `Comparable` 인터페이스를 구현하는 이유가 바로 이것입니다.

`Comparable` 은 원소가 컬렉션에 추가될때 자동적으로 정렬되도록 (*natural sorting*) 하기 위해 사용하고, `Comparator` 는 추가적인 정렬방법을 이용하기 위해 정의할 수 있습니다. [여기](http://www.java2blog.com/2013/02/difference-between-comparator-and.html)서 가져온 예제를 보면

```java
// Comparable
public class Country implements Comparable<Country>{
       @Override
    public int compareTo(Country country) {
        return (this.countryId < country.countryId ) ? -1: (this.countryId > country.countryId ) ? 1:0 ;
}} 

// Comparator

Country indiaCountry=new Country(1, "India");
Country chinaCountry=new Country(4, "China");
Country nepalCountry=new Country(3, "Nepal");
Country bhutanCountry=new Country(2, "Bhutan");
        
List<Country> listOfCountries = new ArrayList<Country>();
listOfCountries.add(indiaCountry);
listOfCountries.add(chinaCountry);
listOfCountries.add(nepalCountry);
listOfCountries.add(bhutanCountry); 

Collections.sort(listOfCountries,new Comparator<Country>() {
  @Override
  public int compare(Country o1, Country o2) {
    return o1.getCountryName().compareTo(o2.getCountryName());
  }
});
```

<br/>

### References

(1) [Useful Java Collection Interview Questions](http://howtodoinjava.com/2013/07/09/useful-java-collection-interview-questions/#why_map_not_extend_collection)  
(2) [Title Image](http://websphereemerge.blogspot.kr/)  
(3) [http://www.java-redefined.com](http://www.java-redefined.com/2014/05/java-collection-interview-questions.html)  
(4) [http://www.java2blog.com/](http://www.java2blog.com/2013/02/difference-between-comparator-and.html)  
(5) [http://www.javatpoint.com](http://www.javatpoint.com/java-collections-interview-questions)  
(6) [SO:  Iterator.remove() vs Collection.remove()](http://stackoverflow.com/questions/14200489/collection-iterator-remove-vs-collection-remove)
