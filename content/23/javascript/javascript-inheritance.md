+++
date = "2016-06-25T12:22:02+09:00"
prev = "../"
title = "Javascript Inheritance"
toc = true
weight = 11
aliases = [
    "/javascript-object-inheritance"
]
+++

## Prototype Inheritance?

자바스크립트는 프로토타입 방식의 상속을 사용한다고들 말합니다. 프로토타입이란 무엇이고, 클래스 기반 상속과는 어떻게 다른지, 그리고 주의해야 할 점은 무엇인지 알아보겠습니다. 이 글에서 다루는 키워드는 아래와 같습니다.

- .constructor
- .\_\_proto\_\_
- .prototype
- Object.create
- new
- Object, Function


먼저 예제부터 보시겠습니다.

```javascript
// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create

// Shape - superclass (1)
function Shape() {
  this.x = 0;
  this.y = 0;
}

// superclass method (2)
Shape.prototype.move = function(x, y) {
    this.x += x;
    this.y += y;
    console.info("Shape moved.");
};

// Rectangle - subclass (3)
function Rectangle() {
  Shape.call(this); // call super constructor.
}

// subclass extends superclass
Rectangle.prototype = Object.create(Shape.prototype); (4)
Rectangle.prototype.constructor = Rectangle; (5)

var rect = new Rectangle();

rect instanceof Rectangle // true.
rect instanceof Shape // true.

rect.move(1, 1); // Outputs, "Shape moved."
```

(1). 먼저 `Shape` 이라는 함수를 만듭니다. 자바스크립트에서는 객체를 생성하기 위한 함수를 `Constructor(생성자)` 라고 부르며, 생성자임을 알아볼 수 있게 첫 글자를 **대문자**로 작성하는 것이 관례입니다. 이렇게 만들어진 생성자는 `new` 를 통해 호출할 수 있습니다. 물론 생성자는 그 자체로서 함수이기 때문에 `Shape()` 과 같이 호출도 가능합니다. 그러나 `new` 가 있는것과 없는것은 조금 차이가 있습니다. 아래에서 알아보겠습니다.

(2). 생성자에 존재하는 `prototype` 속성, 즉 `Shape.prototype` 에 `move` 란 메소드를 추가하고 있습니다. 생성자의 `prototype` 속성에 추가된 모든 것들은 이 생성자를 통해 생성된 모든 객체에서 사용 가능합니다. 그러나, 생성자의 `prototype` 이 아니라, 인스턴스에 추가된 속성들은 그 인스턴스만 사용할 수 있습니다.

```javascript
 var s1 = new Shape();
s1.move(3, 3);

s1.isShape = true;

var s2 = new Shape();
console.log (s2.isShape) // undefined
console.log (typeof s2.isShape === "undefined") // true
```

(3). `Rectangle` 이라는 새로운 생성자를 정의하고 있습니다. 그리고 이 생성자 내에서 `Shape.call(this)` 를 호출하여, `new Rectangle()` 을 통해 만들어지는 모든 인스턴스가 `Shape` 처럼 `this.x` 와 `this.y` 를 가지고, 0으로 초기화 하도록 합니다. 이 과정이 끝나면 다음과 같은 결과가 나옵니다.

```javascript

function Rectangle() {
	Shape.call(this);
}

var r1 = new Rectangle();
console.log (r1.x); // 0
console.log (r2.y); // 0
```

그러나 이 시점에서 `r1` 은 `move` 란 메소드를 사용할 수 없습니다. `Shape.move` 는 있지만 이 `move` 메소드를 상속받은것은 아닙니다. `Rectangle` 은 단지 `this.x`, `this.y` 란 멤버변수를 상속받은 것 뿐입니다.

```javascript
console.log (r1.move); // undefined 
```

(4). 드디어 `Rectangle.prototye` 에 `Shape.prototype` 을 연결해 주어 `r1` 에서도 `Shape.prototype` 에 정의된 메소드들을 사용할 수 있게끔 해줍니다. 


```javascript
r1.move(2, 2);
console.log(r1.x); // 2
console.log(r1.y); // 2
```

여기서 Rectangle.prototype = Shape.prototype 을 하지않고 새롭게 Object.create 해주는 이유는, `Rectangle.prototype` 에 새로운 속성을 추가했을때, `Shape.prototype` 에 추가되도록 하지 않기 위함입니다. 다시 말해, `Rectangle` 에 추가한 것은, `Rectangle` 에만 추가되라는 것이지요.  

`Rectangle.prototype = new Shape();` 처럼 `new` 를 사용하게 되면, 생성자를 호출하게 되어 `Rectangle.prototype` 에 인스턴스 변수인 `this.x` 와 `this.y` 가 추가됩니다.   이것은 우리가 원하지 않은 동작이기에, 일반적으로 `new` 를 이용해서 프로토타입을 생성하지 않습니다.

(5). `Object.create(Shape.prototype)` 으로 생성한 객체는 `constructor` 값으로 `Shape` 를 가지고 있습니다. 이 객체를 이제, `Rectangle.prototype` 에서 사용하므로 값을 변경해 줍니다. 


### __proto__

`__proto__` 속성은 자바스크립트에서 상속의 핵심입니다. 모든 객체들은 자신의 속성을 찾다가 실패하면, `__proto__` 를 통해 더 검색을 시도합니다. 무슨말인고 하니, 다음과 같은 코드가 있을때

```javascript

var r1 = new Rectangle();

r1.move(1, 1);
```

실제로 `Rectangle` 은 `move` 라는 메소드를 인스턴스 멤버로도, 프로토타입 멤버로도 가지고 있지 않습니다. 다시 말해서, 아래와 같은 코드를 작성한 적이 없단 말이지요.

```javascript
// Method per instance
function Rectangle() {
	this.move = function(_x, _y) { this.x = _x; this.y = _y; };
}

// Method for specific instance
r1.move = function(_x, _y) { this.x = _x; this.y = _y; };

// Prototype method
Rectangle.prototype.move = function(_x, _y) { this.x = _x; this.y = _y; };
```

이런 작업을 해 준 적이 없는데, 어떻게 `move` 메소드를 찾는걸까요? 우리는 `move` 를 `Shape.prototype` 에만 추가했는데요! 비결은 아래와 같습니다.

1. `r1` 인스턴스 자체에 `move` 메소드가 인스턴스에 없기 때문에 `r1.__proto__` 에서 탐색하게 됩니다.

2. 인스턴스가 가지고 있는 `__proto__` 의 값은, 생성자의 프로토타입, 즉 `Rectangle.prototype` 입니다. 따라서 이곳을 검색합니다. 그러나 `Rectangle` 프로토타입에도 `move` 메소드는 없습니다.

3. `Rectangle.prototype.__proto__` 를 검색합니다. `Rectangle.prototype` 은 `Object.create(Shape.prototype)` 을 통해 생성되었고, 이것은 인스턴스 멤버가 없는 `Shape` 인스턴스 이기 때문에, `Rectangle.prototype.__proto__` 의 값은 `Shape.prototype` 이 됩니다. 

4. `Shape.prototype` 에는 `move` 가 있기 때문에, 이를 실행합니다.

5. 만약 `Shape.prototype` 에도 `move` 가 없다면, `Shape.prototype.__proto__` 를 탐색합니다. 모든 객체는 Default 값으로 `Object` 를 상속받으며, `Shape` 도 마찬가지입니다. `Shape` 은 `Object` 를 상속받았기 때문에 `Shape.prototype.__proto__` 는 `Object.prototype` 을 가리킵니다. 여기서 메소드를 검색합니다.

6. 만약 `Object.prototype` 에도 없다면, `Object.prototype.__proto__` 를 검색하나, 이 값은 `null` 이기 때문에 멤버 검색에 실패하고 `undefined` 를 돌려줍니다.

다른 예제지만, 이미지를 통해 보는것도 이해에 도움이 될 듯 하여 이미지를 같이 첨부합니다.

<br/>
<p>
<img src="http://mckoss.com/jscript/Prototype.gif" />
</p>
<p align="center">
(http://mckoss.com/jscript/object.htm)
</p>
<br/>


### Object, Function

자바스크립트의 모든 함수는 `Function` 의 인스턴스입니다. 무슨 말인고 하니, 사용자가 정의한 함수들은 `__proto__` 값으로 `Function.prototype` 을 가진다는 뜻이지요.

```javascript

function example() {};

example.__proto__ === Function.prototype; // true
```

그리고 `Function` 은 `Object` 를 상속합니다. 다시 말해, 

```javascript
example.__proto__.__proto__ == Object.prototype
```

그리고 이전에 언급했듯이, `Object.prototype.__proto__` 는 `null` 입니다.

```javascript
Object.prototype.__proto__ === null // true
```

그리고 `Object` 그 자체는, `Function` 을 상속합니다.

```javascript
Object.__proto__ === Function.prototype // true
```

그래서 `Function` 과 `Object` 를 설명할때, 아래와 같은 그림으로 설명할 수 있습니다. 아래 그림에서 빨간 선으로 이어진 `[[Prototype]]` 은 `__proto__` 입니다.

<br/>
<p align="center">
<img src="http://i.stack.imgur.com/rcGmc.png" />
</p>
<p align="center">
(http://iwiki.readthedocs.org/en/latest/javascript/js_core.html#inheritance)
</p>
<br/>


### Prototype Inhertance vs Classical Inheritance

"그래요. 프로토타입 기반 상속이란 이런거군요!. 근데 이거 왜 하는건가요?"

제 짧은 지식으로 어줍잖게 대답하는 것보다, 링크로 연결해드리는게 더 나을것 같아서 관련 링크를 적어놓습니다. 꼭 읽어보셨으면 좋겠습니다.

1. [classical-inheritance-vs-protoypal-inheritance-in-javascript](http://stackoverflow.com/questions/19633762/classical-inheritance-vs-protoypal-inheritance-in-javascript)

2. [why-prototypal-inheritance-matters](http://aaditmshah.github.io/why-prototypal-inheritance-matters/#toc_6)

3. [benefits-of-prototypal-inheritance-over-classical](http://stackoverflow.com/questions/2800964/benefits-of-prototypal-inheritance-over-classical)

4. [classical-vs-prototypal-inheritance](http://stackoverflow.com/questions/1450582/classical-vs-prototypal-inheritance)


### new vs Object.create

위에서 잠깐 언급했듯이 일반적으로는 프로토타입 객체를 만들기 위해서 `Object.create()`를 사용한다고 했었습니다. `new` 대신에요. 왜 그런가 `Object.create` 의 동작을 한번 알아보겠습니다.

1. `Object.create` 는 첫 번째 인자로 프로토타입을 받습니다.
2. 빈 객체를 하나 만들고, 이 객체의 `__proto__` 에 인자로 받은 프로토타입 객체를 연결합니다.
3. 프로토타입이 연결된 객체를 리턴합니다.

아마 코드는 아래와 비슷할 겁니다. 간단한 설명을 위해 두번째 인자는 생략하겠습니다.

```javascript
Object.prototype.create == function(proto) {
  var obj = {};
  obj.__proto__ = proto;
  return obj;
}
```

따라서 어떠한 경우에도 생성자를 호출하지 않으므로 다음과 같은 코드가 생성자에 있을 경우 호출되지 않을겁니다.

```javascript

function Shape() {
  this.x = 0;
  this.y = 0;
  
  console.log("This is constructor for Shape");
}

var created = Object.create(Shape.prototype);
var newed  = new Shape(); // "This is constructor for Shape" 

console.log( created.x ); // undefined
console.log( newed.x ); // 0;
```

`new` 를 이용해 생성한 객체만 생성자가 호출되어, **"This is constructor for Shape"** 가 출력되고 `this.x = 0` 이 실행됩니다. 

`new Shape()` 의 로직은 아마 다음과 비슷할 겁니다. (더 자세한 내용은 [MDN: new Operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/new) 를 보시면 되겠습니다.)

1. 새로운 오브젝트를 생성하고,
2. 이 오브젝트의 `__proto__` 에 생성자의 프로토타입 객체를 연결합니다.
3. 생성자를 호출하고, 리턴합니다.

```javascript
// new Shape();

{
  var obj = {};
  obj.__proto__ = Shape.prototype;
  return Shape.apply(obj, arguments) || obj; 
}
```

이렇게 `new` 연산자는 생성자를 호출하기 때문에, 새롭게 사용할 프로토타입 객체에 의도치 않은 속성이 추가될 수 있습니다. 위의 예에서 `Rectangle` 에서 새롭게 사용할 프로토타입 객체는, 다시 말해 `Rectangle.prototype` 에 들어갈 객체는 단순히 `__proto__` 값으로 `Shape.prototype` 만 가지고 있으면 됩니다. 

만약 `Object.create` 대신 `new` 를 사용하게 되면 `Rectangle.prototype.x` 와 `Rectangle.prototype.y` 가 `0` 으로 초기화되게 됩니다. 이는 원치 않았던 부작용입니다. 이런 이유에서 일반적인 경우에는 `new` 대신 `Object.create` 를 프로토타입 객체 생성에 사용해야 합니다. 아래처럼요

```javascript
// subclass extends superclass
Rectangle.prototype = Object.create(Shape.prototype);
```


### .constructor

모든 프로토타입 객체들은, `constructor` 란 프로퍼티가 있습니다. 이 값은 생성자를 가리킵니다. 그러므로 아래 코드는 `true` 를 출력합니다.

```javascript
Rectangle.prototype.constructor === Rectangle // true
```

우리의 예제인 `Rectangle` 에서도 위에 있는 코드처럼 프로토타입의 `constructor` 값을 초기화 하고 있습니다. 왜 그래야 할까요?

사실 `.constructor` 값은 별로 의미있는 값은 아닙니다. 만약 우리가 `constructor` 값으로 어떤 종류의 객체인지 판별한다면, 의미는 있겠지요. 그러나 일반적으로는 `instanceof` 를 사용합니다.

```javascript
function Shape() { this.x = 0; }

var s1 = new Shape();

console.log( s1.__proto__.constrctor === Shape) // true
console.log( s1 instanceof Shape) // true
```

그럼 이렇게 `constructor` 를 비교하는 작업을 `instanceof` 내부에서 사용하느냐, 그것도 아닙니다. `instanceof` 는 `s1.__proto__` 와 `Shape.prototype` 을 비교합니다.

`.constructor` 는 사실 정말로 쓸모가 없을지도 모르겠습니다. 그러나 자바스크립트 표준이 프로토타입 객체의 `constructor` 프로퍼티는 생성자를 가르켜야 한다고 말하는 한, 적어도 세팅은 해주는게 나쁘지 않다는게 제 생각입니다. 아래는 관련된 논의입니다. 

Link : **[What it the significance of the javascript constructor property](http://stackoverflow.com/questions/4012998/what-it-the-significance-of-the-javascript-constructor-property)**

다시 우리의 예제로 돌아와서, 코드를 살펴보겠습니다.

```javascript
// subclass extends superclass
Rectangle.prototype = Object.create(Shape.prototype); (1)
Rectangle.prototype.constructor = Rectangle; (2)
```

`Rectangle.prototype.constuctor` 를 다시 세팅해 주는 이유는, 이 값이 `Shape` 이기 때문입니다. `Rectangle.protoype` 은 `__proto__` 를 `Shape.prototype` 으로 가지는 오브젝트고, 따라서 (1) 라인에서 코드를 실행시켰을 때는 다음과 같은 결과가 나옵니다.

`console.log( Rectangle.prototype.constructor ); // Shape`

왜냐 하면 `Rectangle.prototype` 에는 `constructor` 가 없기 때문에 `Rectangle.prototype.__proto__` 에서 `constructor` 를 찾는데, `Rectangle.prototype.__proto__` 는 `Shape.prototype` 이기 때문이지요. 

기본적으로 우리가 생성자를 만들면, 자바스크립트는 다음과 같이 프로토타입 객체를 만들고 이 프로토타입 객체의 `constructor` 를 세팅해 줍니다.

```javascript
function Shape() { this.x = 0; }

console.log( Shape.prototype.constructor ); // Shape;
```

`Rectangle.prototype.constructor` 는 본래 처음 `Rectangle` 생성자를 만들었을때는 `Rectangle` 이었겠지만, (1) 라인의 코드를 실행 시킨 순간 `Shape` 으로 변경되고, 더 정확히 이 값은 `Rectangle.prototype.__proto__.consturctor` 에서 옵니다. 결국 값이 바뀌었기 때문에 원래대로 돌려주어야 하므로 아래와 같은 코드를 작성해준 것입니다.

```javascript
Rectangle.prototype.constructor = Rectangle;
```

자 이제, 아래 그림이 완벽히 이해되실 겁니다. 

<br/>
<p align="center">
<img src="http://i.stack.imgur.com/UfXRZ.png" />
</p>
<p align="center">
(http://dmitrysoshnikov.com/ecmascript/javascript-the-core/)
</p>
<br/>
