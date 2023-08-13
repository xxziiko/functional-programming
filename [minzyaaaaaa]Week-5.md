
# Capter 10 - 11. 일급 함수

<aside>
💡 일급 함수와 고차 함수

</aside>

## 코드의 냄새 제거하기

Code Smell : 함수 이름에 있는 암묵적 인자 

특징 

1. 거의 똑같이 구현된 함수가 있다.
2. 함수 이름이 구현에 있는 다른 부분을 가리킨다.

### 리팩토링 방법

**암묵적 인자를 드러내기**

암묵적 인자가 일급 값이 되도록 함수에 인자를 추가하는 방법. 잠재적 중복을 없애고 코드의 목적을 더 잘 표현할 수 있다.

단계

1. 함수 이름에 있는 암묵적 인자를 확인한다.
2. 명시적인 인자를 추가한다.
3. 함수 본문에 하드 코딩된 값을 새로운 인자로 바꾼다.
4. 함수를 호출하는 곳을 고친다.

**함수 본문을 콜백으로 바꾸기**

원래 있던 코드를 고차 함수로 만드는 강력한 방법이다.

단계

1. 함수 본문에서 바꿀 부분의 앞부분과 뒷부분을 확인한다.
2. 리팩토링 할 코드를 함수로 빼낸다.
3. 빼낸 함수의 인자로 넘길 부분을 또 다른 함수로 빼낸다.

- 예시
    
    ```jsx
    // 암묵적 인자를 드러내기
    // Pirce가 암묵적 인자 
    function setPriceByName(cart, name, price) {
    	var item = cart[name];
    	var newItem = objectSet(item, 'price', price);
    	var newCart = objectSet(cart, name, newItem);
    	return newCart;
    }
    
    cart = setPriceByName(cart, 'shoe', 13);
    cart = setQuantityByName(cart, 'shoe', 3);
    cart = setShippingByName(cart, 'shoe', 0);
    cart = setTaxByName(cart, 'shoe', 2.34);
    ```
    
    ```jsx
    // 명시적인 인자 추가하기
    function setFieldByName(cart, name, field, value) {
    	var item = cart[name];
    	var newItem = objectSet(item, field, value);
    	var newCart = objectSet(cart, name, newItem);
    	return newCart;
    }
    
    cart = setFieldByName(cart, "shoe", 'price', 13);
    cart = setFieldByName(cart, "shoe", 'quantity', 13);
    cart = setFieldByName(cart, "shoe", 'shipping', 13);
    cart = setFieldByName(cart, "shoe", 'tax', 13);
    
    // 값에는 큰 따옴표를 쓰고 키에는 작은 따옴표를 썼다.
    // 만약 'shoe'처럼 키로도 쓰고 값으로도 쓴다면 큰따옴표를 사용한다.
    ```
    
    ```jsx
    // 원래 API 사용법
    function setPriceByName(cart, name, price)
    function setQuantityByName(cart, name, quant)
    function setShippingByName(cart, name, ship)
    function setTaxByName(cart, name, tax)
    
    // 새로운 API 사용법
    function setFieldByName(cart, name, field, value);
    // field 목록 - 'price', 'quantity', 'shipping', 'tax'
    ```
    

## 일급 값

- 함수에 인자로 넘길 수 있다.
- 함수의 리턴값으로 받을 수 있다.
- 변수에 할당할 수 있다.
- 배열이나 객체의 항목으로 넣을 수 있다.
- 자바스크립트에서 일급이 아닌 것

숫자는 함수에 인자로 넘길 수 있고 함수의 리턴값으로 받을 수 있습니다. 또 변수에 넣을 수 있고 배열이나 객체의 항목으로 넣을 수도 있습니다. 문자열이나 불리언값, 배열, 객체도 비슷하게 할 수 있습니다. 자바스크립트나 다른 많은 언어에서 함수 역시 비슷하게 쓸 수 있습니다. 이런 식으로 쓸 수 있는 값을 일급이라고 합니다.

자바스크립트에서 일급이 아닌것 - 수식 연산자, 반복문, 조건문, try/catch 블록

## 컴파일, 런타임

- 컴파일 타임에 검사하는 방법 - 정적 타입 시스템 (typescript)
- 런타임에 검사하는 방법 - 함수를 실행할 때마다 동작 (javascript)
- 런타임 검사
    
    ```jsx
    // 런타임 검사 방법
    var validItemFields = ['price', 'quantity', 'shipping', 'tax'];
    
    function setFieldByName(cart, name, field, value) {
    	if(!validItemFields.includes(field)) {
    		throw "Not a valid item field: " +
    					"'" + field + "'.";
    	}
    	var item = cart[name];
    	var newItem = objectSet(item, field, value);
    	var newCart = objectSet(cart, name, newItem);
    	return newCart;
    }
    ```
    

## 일급 필드를 사용하면 API를 바꾸기 더 어렵나요?

엔티티(entity) 필드명을 일급으로 만들어 사용하는 것이 세부 구현을 밖으로 노출하는 것이 아닐까?

필드명은 계속 유지해야 한다. 하지만 구현이 외부에 노출된 것은 아니다. 만약 내부에서 정의한 필드명이 바뀐다고 해도 사용하는 사람들이 원래 필드명을 그대로 사용하게 할 수 있다.

예를 들어 ‘quantity’ 필드명이 어떤 이유로 ‘number’라는 이름으로 바뀌었다고 생각하면, 코드 전체를 바꾸지 않고 추상화 벽에서 ‘quantity’ 필드명을 그대로 사용하고 싶다면 내부에서 간단히 필드명을 바꿔 주면 된다.

```jsx
var validItemFields = ['price', 'quantity', 'shipping', 'tax', 'number'];
var translations = { 'quantity': 'number' };

function getFieldByName(cart, name, fields, value) {
	if(!validItemFields.includes(field)){
		throw "Not a valid item field: '" + field + "'.";
		if(translations.hasOwnProperty(field)){
			field = translations[field];
		}
		var item = cart[name];
		var newItem = objectSet(item, field, value);
		var newCart = objectSet(cart, name, newItem);
		return newCart;
	}
}
```

이런 방법은 필드명이 일급이기 때문에 할 수 있는 것이다. 필드명이 일급이라는 말은 객체나 배열에 담을 수 있다는 뜻입니다. 그리고 언어에 모든 기능을 이용해서 필드명을 처리할 수 있다.

- 일급 필드
    
    일급 필드(First-class field)는 객체 내의 필드(멤버 변수)가 함수와 동일한 권한과 기능을 갖는 것을 의미한다. 즉, 필드를 일반적인 변수나 매개변수처럼 다룰 수 있다.
    
    ```jsx
    class Person {
    	name = "John" // 일급 필드
    	age = 20
    
    	sayHello() {
    		console.log('Hello, my name is ${this.name}`);
    	}
    	getName() {
    		console.log(name);
    	}
    	getField(field) {
    		console.log(field);
    	}
    }
    
    const Person = new Person();
    person.sayHello(); // Hello, my name is John
    person.getName(); // John
    person.getField(name);
    ```
    
    위의 코드에서 `name`은 `Person`의 클래스에 일급 필드로 선언되었다. 필드는 클래스 내부에서 직접 값을 가지며, 객체를 생성할 때마다 해당 값이 할당된다. `sayHello()` 메서드는 필드에 접근해서 값을 사용할 수 있다.
    
    일급 필드를 사용하여 객체의 상태를 보다 명확하게 표현할 수 있고, 코드의 가독성이 향상된다. 또한, 필드를 클래스 내부에서 캡슐화할 수 있어서 외부에서의 무분별한 접근을 제어할 수 있다. 또한, 일급 필드는 클래스의 인스턴스마다 고유한 값을 가질 수 있으므로, 클래스의 다양한 인스턴스 간에 다른 상태를 유지할 수 있다. 
    
    일급 필드는 객체 지향 프로그래밍에서의 캡슐화와 불변성 원칙을 강화하고, 코드를 더 깔끔하고 유연하게 작성할 수 있도록 도와준다.
    

## 데이터 지향 (data orientation)

객체나 배열을 너무 많이 쓰게 된다.

데이터를 데이터 그대로 사용하는 것의 중요한 장점은 여러 가지 방법으로 해석할 수 있다는 점이다. 제한된 API로 정의하면 데이터를 제대로 활용할 수 없다. 

데이터 지향 (data orientation)은 이벤트와 엔티티에 대한 사실을 표현하기 위해 **일반 데이터 구조**를 사용하는 프로그래밍 형식이다.

> 데이터의 흐름에 중점을 두어 작성하는 방식을 의미한다. 반대로 명령형 프로그래밍에서는 프로그램의 상태를 변경하는 명령문의 순서에 초점을 맞추는 것과 대조된다.
> 

## 고차 함수(higer-order function)

함수가 다른 함수를 인자로 받을 수 있다.

코드를 추상화 하기 용이하다.

**리팩토링 단계**

1. 코드를 함수로 감싸기
2. 더 일반적인 이름으로 바꾸기
3. 암묵적 인자를 드러내기 
4. 함수 추출하기 
5. 암묵적 인자를 드러내기

# Summary

- 일급 값은 변수에 저장할 수 있고 인자로 전달하거나 함수의 리턴값으로 사용할 수 있다. 일급 값은 코드로 다룰 수 있는 값이다.
- 언어에는 일급이 아닌 기능이 많이 있다. 일급이 아닌 기능은 함수로 감싸 일급으로 만들 수 있다.
- 어떤 언어는 함수를 일급 값처럼 쓸 수 있는 일급 함수가 있다. 일급 함수는 어떤 단계 이상의 함수형 프로그래밍을 하는 데 필요하다.
- 고차 함수는 다른 함수에 인자로 넘기거나 리턴값을 받을 수 있는 함수이다. 고차 함수로 다양한 동작을 추상화할 수 있다.
- 함수 이름에 있는 암묵적 인자는 함수의 이름으로 구분하는 코드의 냄새이다. 이 냄새는 코드로 다룰 수 없는 함수 이름 대신 일급 값인 인자로 바꾸는 암묵적 인자를 드러내기 리팩토링을 적용해서 없앨 수 있따.
- 동작을 추상하기 위해 본문을 콜백으로 바구기 리팩토링을 사용할 수 있다. 서로 다른 함수의 동작 차이를 일급 함수 인자로 만든다.

## 일급 함수

함수를 리턴하는 함수

**Rest Argument**

함수 정의에서 매개변수 목록 내에 `…`을 사용하여 나타낼 수 있는 매개변수

이는 함수에 전달되니 인수의 잔여 부분을 배열로 수집하는데 사용된다. 

여러 개의 인수를 받을 수 있고, 함수 내에서 배열로 처리할 수 있다.

- 예시
    
    ```jsx
    function sum(...numbers) {
    	let total = 0;
    	for (let number of numbers) {
    		total += number;
    	}
    	return total;
    }
    
    console.log(sum(1, 2, 3, 4)); // 10
    ```
    

**Spread Operator**

# Summary

- 일급 값과 일급 함수,  고차함수
- 고차 함수로 패턴이나 원칙을 코드로 만들 수 있다. 고차 함수를 사용하지 않는다면 일일이 수작업을 해야 한다. 고차 함수는 한 번 정의하고 필요한 곳에 여러 번 사용할 수 있다.
- 고차 함수로 함수를 리턴하는 함수를 만들 수 있다. 리턴 받은 함수는 변수에 할당해서 이름이 있는 일반 함수처럼 쓸 수 있다.
- 고차 함수를 사용하면서 잃는 것도 있다. 고차 함수는 많은 중복 코드를 없애 주지만 가독성을 해칠 수고 있다.
