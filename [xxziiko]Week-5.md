# Chapter 10.

- 일급 값?
- 문법을 일급 함수로 만드는 방법
- 고차함수
- 일급 함수와 고차함수를 사용한 리팩토링

- ## 코드의 냄새: 함수 이름에 있는 암묵적 인자

- 코드의 냄새 = 함수 이름에 암묵적 인자가 있을 때

```jsx
function setPriceByName(cart, name, price) {
 const item = cart[name];
 const newItem = objectSet(item, 'price', price);
 const newCart = objectSet(cart, name, newItem);
 return newCart;
}

function setShippingByName(cart, name, ship) {
 const item = cart[name];
 const newItem = objectSet(item, 'shipping', ship);
 const newCart = objectSet(cart, name, newItem);
 return newCart;
}
```

- **함수 이름에 있는 암묵적 인자** 냄새는 두 가지 특징을 가짐
    - 함수 구현이 거의 동일하다
    - 함수 이름이 구현의 차이를 만든다
    

## 리팩터링: 암묵적 인자를 드러내기

- 함수 이름의 일부가 암묵적 인자로 사용되고 있다면 **암묵적 인자를 드러내기**(express implicit argument) 리팩터링을 사용할 수 있음
- 드러낸다 → 암묵적인 것을 명시적으로 바꾼다
- 리팩토링 단계
    1. 함수 이름에 있는 암묵적 인자를 확인한다.
    2. 명시적인 인자를 추가한다.
    3. 함수 본문에 하드 코딩된 값을 새로운 인자로 바꾼다.
    4. 함수를 부르는 곳을 고친다.

```jsx
// 리팩터링 전
function setPriceByName(cart, name, price) {
 const item = cart[name];
 const newItem = objectSet(item, 'price', price);
 const newCart = objectSet(cart, name, newItem);
 return newCart;
}

// 리팩터링 후
function setFieldByName(cart, name, field, value) {
 const item = cart[name];
 const newItem = objectSet(item, field, value);
 const newCart = objectSet(cart, name, newItem);
 return newCart;
}
```

- 리팩터링으로 필드명을 **일급 값**으로 만들었음.
- **일급 값**은 언어에 있는 다른 값처럼 쓸 수 있다.

## 일급인 것과 일급이 아닌 것을 구별하기

**자바스크립트에서 일급이 아닌 것**

1. 수식 연산자
2. 반복문
3. 조건문
4. `try/catch` 블록

**일급으로 할 수 있는 것**

1. 변수에 할당
2. 함수의 인자로 넘기기
3. 함수의 리턴값으로 받기
4. 배열이나 객체에 담기

## 일급 값 검사하기

1. 컴파일 타임에 검사
    
    → 타입 검사기가 코드를 실행하기 전에 알려줌
    
2. 런타임에 검사

```jsx
const validItemFields = ['price', 'quantity', 'shipping', 'tax'];

function setFieldByName(cart, name, field, value) {
 if(!validItemFields.includes(field)) throw 'Not a valid item field: ' + field;

 const item = cart[name];
 const newItem = objectSet(item, field, value);
 const newCart = objectSet(cart, name, newItem);
 return newCart;
}
```

## API 문서에서 필드명이 변경됐을 때

```jsx
const validItemFields = ['price', 'quantity', 'shipping', 'tax'];
const translations = { 'quantity': 'number'};

function setFieldByName(cart, name, field, value) {
 if(!validItemFields.includes(field)) throw 'Not a valid item field: ' + field;
 if(translations.hasOwnProperty(field)) field = translations[field];

 const item = cart[name];
 const newItem = objectSet(item, field, value);
 const newCart = objectSet(cart, name, newItem);
 return newCart;
}
```

- 필드명이 일급이기 때문에 가능한 것
- 필드명이 일급이라는 것은 객체나 배열에 담을 수 있다는 뜻

## 연습 문제

**연습문제1.**

```jsx
function multiply (x , number) {
	return x * number
}
```

**연습문제2.**

```jsx
function incrementFieldByName (cart, name, field) {
	const item = cart[name];
	const value = item[field];
	const newValue = value + 1;
	const newItem = ObjectSet(item, field, newValue);
	const newCart = ObjectSet(cart, name, newItem);
	return newCart;
	
}
```

### 데이터 지향

- 이벤트와 엔티티에 대한 사실을 표현하기 위해 일반 데이터구조를 사용하는 프로그래밍 형식

### 정적 타입 vs 동적 타입

- 정적 타입: 컴파일 할 때 타입 검사를 하는 언어
- 동적 타입: 컴파일 할 때 타입 검사를 하지 않지만 런타임에 타입을 확인하는 언어

## 반복문 일급함수 만들기

```jsx

function operateOnArray (array, f) {
	for(let i = 0; i < array.length; i ++) {
		const item = array[i];
		f(item);
	}
}

function cookAndEat (food) {
	cook(food);
	eat(food);
}

function clean (dish) {
	wash(dish);
	dry(dish);
	putAaway(dish);
}

operateOnArray(foods, cookAndEat);
operateOnArray(dishes, clean);
```

- `operateOnArray()` 함수는 `forEach()` 내장 함수와 기능이 같다.

```jsx
function forEach(array, f) {
	for(let i = 0; i < array.length; i ++) {
		const item = array[i];
		f(item);
	}
}

forEach(foods, cookAndEat);
forEach(dishes, clean);
```

- `forEach()` 함수는 배열과 함수를 인자로 받음 → 고차 함수
- 고차함수는 인자로 함수를 받거나 리턴 값으로 함수를 리턴할 수 있음

- 익명함수 사용하기(인라인 함수)

```jsx
forEach(foods, function(food) {
	cook(food);
	eat(food);
});

forEach(dishes, function(dish) {
	wash(dish);
	dry(dish);
	putAaway(dish);
});
```

- 자바스크립트에서 인자로 전달하는 함수를 **콜백**이라고 부름
- 콜백함수는 나중에 호출 될 것을 기대
- 핸들러 함수라고도 함

## 함수를 정의하는 방법

1. 전역으로 정의

```jsx
function saveCurrentUserData () {
	saveUserData(user);
}

withLogging(saveCurrentUserData);
```

1. 지역적으로 정의

```jsx
function someFunction () {
	const saveCurrentUserData = () => {
		saveUserData(user);
	};

	withLogging(saveCurrentUserData);
}
```

1. 인라인으로 정의

```jsx
withLogging(function() {saveUserData(user);});
```

## 왜 본문을 함수로 감싸나요?

- 위의 코드에서 함수를 만들어 감싼 이유는 코드가 바로 실행되지 않게 하기 위해서임
- 실행을 미루고 특정 함수 혹은 문맥 안에서 실행되게 하는 것을 의도할 때 본문을 함수로 감싼다.

## 요점 정리

- 일급 값은 변수에 저장할 수 있고 인자로 전달하거나 함수의 리턴값으로 사용할 수 있음.
- 언어에는 일급이 아닌 기능이 많이 있다. 일급이 아닌 기능은 함수로 감싸 일급으로 만들 수 있음
- 어떤 언어는 함수를 일급 값처럼 쓸 수 있음. 일급 함수는 어떤 단계 이상의 함수형 프로그래밍을 하는 데 필요함
- 고차 함수는 다른 함수에 인자로 넘기거나 리턴값으로 받을 수 있는 함수 → 다양한 동작을 추상화 할 수 있음
- **함수에 이름이 있는 암묵적 인자** 는 함수의 이름으로 구분하는 코드의 냄새 → 코드로 다룰 수 없는 함수 이름 대신 일급 값인 인자로 바꾸는 **암묵적 인자**를 드러내기 리팩터링을 적용해 없앨 수 있다.
- 동작을 추상화하기 위해 **본문을 콜백으로 바꾸기** 리팩터링을 사용할 수 있음 → 서로 다른 함수의 동작 차이를 일급 함수 인자로 만든다.

### 일급함수?

프로그래밍 언어에서 어떤 대상이 일급이라는 것은 그 대상이 다음 세가지 조건을 만족할 때 라고 한다.

1. 함수 호출의 인자로 사용될 수 있다.
2. 함수의 리턴값으로 사용될 수 있다.
3. 대입 연산을 통해서 변수가 가르키는 대상으로 지정될 수 있다.(변수나 데이터에 할당)

- MDN 에서의 일급함수 정의

<aside>
💡 함수를 다른 변수와 동일하게 다루는 언어는 일급 함수를 가졌다고 표현한다. 예를 들어, 일급 함수를 가진 언어에서는 함수를 다른 함수에 매개변수로 제공하거나, 함수가 함수를 반환할 수 있으며, 변수에도 할당할 수 있다.

</aside>

# Chapter 11.

- 함수 본문을 콜백으로 바꾸기 리팩터링
- 함수를 리턴하는 함수가 가진 힘
- 고차함수에 익숙해지기

## 카피-온-라이트 리팩터링하기

- **함수 본문을 콜백으로 바꾸기** 리팩터링 적용

```jsx
function arraySet(array, idx, value) {
	const copy = array.slice();
	copy[idx] = value;
	return copy;
}

function arraySet(array, idx, value) {
	return withArrayCopy(array, (copy) => {
		copy[idx] = value;
	});
}

function withArrayCopy (array, modify) {
	const copy = array.slice();
	modify(copy);
	return copy;
}
```

- 똑같은 코드를 여기저기 만들지 않아도 됨
- `withArrayCopy()` 함수로 기본 연산 뿐만 아니라 배열을 바꾸는 어떠한 동작에도 사용이 가능해짐 (재사용성 상승)
- `withArrayCopy()` 함수는 동작을 최적화하기 더 쉬움
    - 기존 카피-온-라이트 함수는 실행할때마다 새로운 복사본을 만들었지만 `withArrayCopy()` 함수로 복사본 하나만 만들어서 쓸 수 있게 됨.

## 연습문제

- `push()`, `drop_last()`, `drop_first()` 적용해보기

```jsx
function push(array, elem) {
	return withArrayCopy(array, (copy) => {
		copy.push(elem);
	}
}

function drop_last(array) {
	return withArrayCopy(array, (copy) => {
		copy.pop();
	}
}

function drop_first(array) {
	return withArrayCopy(array, (copy) => {
		copy.shift();
	}
}
```

- 객체에 적용해보기

```jsx
function withObjectCopy (object, modify) {
	const copy = object.assing({}, object);
	modify(copy);
	return copy;
}

function objectSet(object, key, value) {
	return withObjectCopy(object, (copy) => {
		copy[key] = value;
})
}

function objectDelete(object, key) {
	return withObjectCopy(object, (copy) => {
		delete copy[key];
})
}
```

- `try/catch` 문에 적용해보기

```jsx
function tryCatch (f, errorHandler) {
	try {
		return f();
	} catch(error) {
		return errorHandler(error);
	}
}
```

## 요점 정리

- 고차 함수를 이용해서 패턴이나 원칙을 코드로 만들 수 있다. → 한번 정의하고 필요한 곳에 여러번 사용 가능 → 재사용성 높아짐
- 고차 함수를 이용해서 함수를 리턴하는 함수를 만들 수 있다. → 리턴 받은 함수는 변수에 할당해서 이름이 있는 일반 함수처럼 사용  할 수 있음
- 고차 함수를 사용하면 중복 코드를 없애주지만 가독성을 해칠 수 있다. → 적절하게 사용
