# Chapter 6.

- 카피-온-라이트 적용방법
- 배열과 객체에서 쓸 수 있는 카피-온-라이트 동작 만들기
- 깊이 중첩된 데이터에서 카피-온-라이트 동작

<br/>
<br/>

### 쓰기 동작은 불변성 원칙에 따라 구현해야 한다.

- 불변성 원칙은 카피-온-라이트라고 한다
- 하스켈이나 클로저 같은 언어도 이 원칙을 사용
- 자바스크립트는 변경 가능한 데이터 구조를 사용하기 때문에 불변성 원칙을 직접 구현해야 함

<br/>
<br/>

## 카피-온-라이트 원칙 세 단계

1. 복사본 만들기
2. 복사본 변경하기(원하는 만큼)
3. 복사본 리턴하기

<aside>
‼️ 카피-온-라이트는 **쓰기**를 **읽기**로 바꾼다.

</aside>

### 예시

**원래 코드**

```jsx
function removeItem(array, idx, count) {
	array.splice(idx, count);
}

function remove_item_by_name(cart, name){
	const new_cart = cart.slice();
	let idx = null;

	for(let i = 0; i < new_cart.length; i ++) {
		if(new_cart[i].name === name) idx = i;
	}

		if(idx !== null) removeItem(new_cart, idx, 1);
		
		return new_cart;
}

```

**카피-온-라이트 적용 코드**

```jsx
function removeItem(array, idx, count) {
	const copy = array.slice();
	copy.splice(idx, count);
	return copy;
}

function remove_item_by_name(cart, name){
	let idx = null;

	for(let i = 0; i < cart.length; i ++) {
		if(cart[i].name === name) idx = i;
	}

		if(idx !== null) return removeItem(cart, idx, 1);
		
		return cart;	
}
```

- 이렇게 적용하면 값을 바꿀 때만 복사한다

<br/>
<br/>

### 연습문제

```jsx
let mailing_list =[];

function add_contact(email) {
	mailing_list.push(email);
}

function submit_from_handler(event) {
	let form = event.target;
	let email = form.elements['email'].value;
	add_contact(email);
}

// 카피-온-라이트
function add_contact(list, email) {
	const	mailing_list_copy = list.slice();
	mailing_list_copy

.push(email)
	return mailing_list
}

function submit_from_handler(event) {
	let form = event.target;
	let email = form.elements['email'].value;
	mailing_list = add_contact(mailing_list, email);
}

```
<br/>
<br/>

## 읽기와 쓰기를 같이 하는 동작은 어떻게 해야 할까?

### 두 가지 접근 방법

1. 함수를 분리하기(읽기 함수, 쓰기 함수) → 더 좋은 방법
2. 값을 두 개 리턴하기

### 1. 함수를 분리하기

- `shift()` 가 리턴하는 값은 배열의 첫 번째 항목 → 배열에 첫 번째 항목을 리턴하는 계산 함수를 만든다.
    
    ```jsx
    function first_element(array) {
    	return array[0];
    }
    ```
    
    - 읽기 함수
    
- `shift()` 의 쓰기 동작만 사용하고 리턴 값은 무시한다.
    
    ```jsx
    function drop_first(array) {
    	array.shift();
    }
    ```
    
    - 쓰기 함수

- 쓰기 동작을 카피-온-라이트로 변경
    
    ```jsx
    function drop_first(array) {
    	const array_copy = array.slice();
    	array_copy.shift();
    	return array_copy;
    }
    ```
    

### 2. 값을 두 개 리턴하는 함수로 만들기

- `shift()`메서드를 바꿀 수 있도록 새로운 함수로 감싼다.
- 읽기와 쓰기를 함께 하는 함수를 읽기만 하는 함수로 바꾼다.
    
    ```jsx
    
    function shift(array) {
    	const array_copy = array.slice();
    	const first = array_copy.shift();
    	return {
    		first : first,
    		array : array_copy
    	};
    }
    ```
    
    **다른 방법**
    
    ```jsx
    function shift(array) {
    	return {
    		first : first_element(array),
    		array : drop_first(array)
    	};
    }
    ```

<br/>
<br/>

### 연습 문제(`pop()`)

1. 읽기 함수와 쓰기 함수로 분리하기
    
    ```jsx
    function last_element(array) {
    	return array[array.length -1];
    }
    
    function drop_last(array) {
    	const array_copy = array.slice();
    	array_copy.pop();
    	return array_copy;
    }
    ```
    

2. 값 두 개를 리턴하는 함수로 만들기

    ```jsx
    function pop(array) {
	    const array_copy = array.slice();
	    const last = array_copy.pop();
		    return {
		    last : last,
		    array : array_copy
	    };
    }
    ```

3. `add_contact()` 리팩토링
    
    **기존 코드**
    
    ```jsx
    function add_contact(mailing_list, email) {
    	const list_copy = mailing_list.slice();
    	list_copy.push(email);
    	return list_copy;
    }
    ```
    
    **답**
    
    ```jsx
    function push(array, element) {
    	const copy = array.slice();
    	copy.push(element);
    	return copy;
    }
    
    function add_contact(mailing_list, email) {
    	list_copy.push(mailing_list, email);
    	return list_copy;
    }
    ```
    
4. 배열 항목을 카피-온-라이트 방식으로 설정
    
    ```jsx
    a[15] = 2;
    
    function arraySet(array, idx, value) {
    	const copy = array.slice();
    	return copy[idx] = value;
    }
    ```
    
<br/>
<br/>

## 불변 데이터 구조를 읽는 것은 계산이다

1. 변경 가능한 데이터를 읽는 것은 액션이다.
2. 쓰기는 데이터를 변경 가능한 구조로 만든다.
3. 어떤 데이터에 쓰기가 없다면 데이터는 변경 불가능한 데이터이다.
4. 불변 데이터 구조를 읽는 것은 계산이다.
5. 쓰기를 읽기로 바꾸면 코드에 계산이 많아진다.

<br/>
<br/>

## 객체에 대한 카피-온-라이트

1. 복사본 만들기
2. 복사본 변경하기
3. 복사본 리턴하기

- `Object.assign()` 함수를 사용하여 카피-온-라이트
    
    ```jsx
    function setPrice (item, new_price) {
    	const item_copy = Object.assign({}, item);
    	 item_copy.price = new_price;
    	 return item_copy;
    }
    ```
    
<br/>
<br/>

### 연습문제

- 카피-온-라이트 방식으로 객체에 값을 설정하는 `objectSet()` 함수
    
    ```jsx
    o['price'] = 37;
    
    function objectSet(object, key, value) {
    	const object_copy = Object.assign({}, object);
    	object_copy[key] = value;
    	return object_copy;
    }
    ```
    

- `objectSet()` 이용하여 `setPrice()` 리팩토링
    
    ```jsx
    function setPrice (item, new_price) {
    	return objectSet(item, 'price', new_price);
    }
    ```
    
- `objectSet()` 함수를 이용하여 제품 개수를 설정하는 `setQuantity()` 함수
    
    ```jsx
    function setQuantity (item, new_quantity) {
    	return objectSet(item, 'quantity', new_quantity);
    }
    ```
    
- 객체의 키로 키/값 쌍을 지우는 delete 연산을 카피-온-라이트 버전으로 만들기
    
    ```jsx
    function objectDelete (object, key) {
    	const copy = Object.assign({}, object);
    	delete copy[key]
    	return copy;
    }
    ```
    
<br/>
<br/>

## 중첩된 쓰기를 읽기로 바꾸기

- 가장 안쪽에 있는 쓰기 동작부터 바꾼다.
    
    ```jsx
    function setPriceByName(cart, name, price) {
    	const cartCopy = cart.slice();
    	for(let i = 0; i < carCopy.length; i++) {
    		if(cartCopy[i].name === name) cartCopy[i] = setPrice(cartCopy[i], price);
    	}
    
    	return cartCopy;
    }
    ```
    
    - 배열에 항목은 바뀌지 않지만 배열 항목이 참조하는 값이 바뀌게 되는 것은 불변 데이터가 아니다.
    - 중첩된 모든 데이터 구조가 바뀌지 않아야 불변 데이터라고 할 수 있다.
    
<br/>
<br/>

## 요점 정리

- 함수형 프로그래밍에서는 불변 데이터가 필요하다. 계산에서는 변경 가능한 데이터에 쓰기를 할 수 없다.
- 카피-온-라이트는 데이터를 불변형으로 유지할 수 있는원칙이다.
- 카피-온-라이트는 값을 변경하기 전에 얕은 복사를 하고 리턴한다.
- 보일러 플레이트 코드를 줄이기 위해 기본적인 배열과 객체 동작에 대한 카피-온-라이트 버전을 만들어 두는 것이 좋다.


<br/>
<br/>

# Chapter 7.

- 레거시 코드나 신뢰할 수 없는 코드로부터 내 코드를 보호하기 위한 방어적 복사
- 얕은 복사와 깊은 복사 비교
- 카피-온-라이트와 방어적 복사를 언제 사용할까

<br/>
<br/>
<br/>


## 우리가 만든 카피-온-라이트 코드는 신뢰할 수 없는 코드와 상호작용 해야한다

- 카피-온-라이트 코드는 불변성이 지켜지는 **안전지대**에 있다.
- 하지만 안전지대 밖으로 나가는 데이터는 잠재적으로 바뀔 수 있다 → 신뢰할 수 없는 코드가 데이터를 바꿀 수 있기 때문
- 신뢰할 수 없는 코드에서 안전지대로 들어오는 데이터도 잠재적으로 바뀔 수 있다.  → 신뢰할 수 없는 코드가 계속 데이터 참조를 가지고 있기 때문
- **방어적 복사**를 통해 데이터가 바뀌는 것을 막아주어야 한다.

<br/>
<br/>

## 방어적 복사 구현

- 인자로 들어온 값이 변경될 수도 있는 함수를 사용하면서 불변성은 지켜야 한다.
- 방어적 복사를 사용하여 데이터가 바뀌는 것을 막아 불변성을 지킨다.

- 데이터를 전달하기 전에 복사
    
    ```jsx
    	function add_item_to_cart (name, price) {
    	const item = make_cart_item(name, price);
    	shopping_cart = add_item(shopping_cart, item);
    	
    	const total = calc_total(shopping_cart);
    	set_cart_total_dom(total);
    	update_shipping_icons(shopping_cart);
    	update_tax_dom(total);
    	
    	const cart_copy = deepCopy(shopping_cart);
    	black_friday_promotion(cart_copy);
    }
    ```
    

- 데이터를 전달하기 전후에 복사
    
    ```jsx
    function add_item_to_cart (name, price) {
    	const item = make_cart_item(name, price);
    	shopping_cart = add_item(shopping_cart, item);
    	
    	const total = calc_total(shopping_cart);
    	set_cart_total_dom(total);
    	update_shipping_icons(shopping_cart);
    	update_tax_dom(total);
    	
    	const cart_copy = deepCopy(shopping_cart);
    	black_friday_promotion(cart_copy);
    	shopping_cart = deepCopy(cart_copy);
    }
    ```
    
<br/>
<br/>

## 방어적 복사 규칙

1. 데이터가 안전한 코드에서 나갈 때 복사하기
    1. 불변성 데이터를 위한 깊은 복사본을 만든다.
    2. 신뢰할 수 없는 코드로 복사본을 전달한다.
    
2. 안전한 코드로 데이터가 들어올 때 복사하기
    1. 변경될 수 도 있는 데이터가 들어오면 바로 깊은 복사본을 만들어 안전한 코드로 전달한다.
    2. 복사본을 안전한 코드에서 사용한다.

- 첫번째 규칙과 두번째 규칙은 순서와 관계없이 사용할 수 있다.
    - 먼저 데이터가 나가고 나중에 들어오는 경우 : 신뢰할 수 없는 라이브러리 함수 사용할 때
    - 먼저 데이터가 들어오고 나중에 나가는 경우: 공유 라이브러리를 만들 때

<br/>
<br/>

## 신뢰할 수 없는 코드 감싸기

- 함수의 가독성과 재사용을 위해 방어적 복사 코드를 분리해 별도의 함수로 만들어 둔다.

- 안전하게 분리한 버전
    
    ```jsx
    function add_item_to_cart (name, price) {
    	const item = make_cart_item(name, price);
    	shopping_cart = add_item(shopping_cart, item);
    	
    	const total = calc_total(shopping_cart);
    	set_cart_total_dom(total);
    	update_shipping_icons(shopping_cart);
    	update_tax_dom(total);
    	
    	shopping_cart = black_friday_promotion_safe(shopping_cart);
    }
    
    function black_friday_promotion_safe(cart) {
    	const cart_copy = deepCopy(cart);
    	black_friday_promotion(cart_copy);
    	return deepCopy(cart_copy);
    }
    ```
    
<br/>
<br/>

### 연습문제

- 연습문제 1
    
    ```jsx
    function payrollCalcSafe(employees) {
    	const employees_copy = deepCopy(employees);
    	payrollChecks = payrollCalc(employees_copy);
    	return deepCopy(payrollChecks);
    }
    ```
    

- 연습문제 2
    
    ```jsx
    userChanges.subscribe(function(user) {
    	const user_copy = deepCopy(user);
    	processUser(user_copy);
    });
    ```
    

<br/>
<br/>

## 방어적 복사가 익숙할 수도 있다

방어적 복사가 사용된 예시

1. 웹 API 속 방어적 복사
    - 대부분의 웹 기반 API는 암묵적으로 방어적 복사를 한다.
        - JSON 데이터의 `request` → 깊은 복사본
        - JSON 데이터의 `response` → 깊은 복사본

1. 얼랭과 엘릭서에서 방어적 복사
    - 얼렝에서 두 프로세스가 서로 메세지를 주고 받을때, 프로세스에서 데이터가 나갈때 방어적 복사본 사용 → 방어적 복사는 얼랭 시스템이 고가용성을 보장하는 핵심 기능

<br/>
<br/>

## 카피-온-라이트와 방어적 복사 비교

**카피-온-라이트**

- 통제할 수 있는 데이터를 바꿀 때
- 안전지대 어디서나 쓸 수 있음. 카피-온-라이트가 불변성을 가진 안전지대를 만든다.
- 얕은 복사
- 규칙
    - 바꿀 데이터의 얕은 복사를 만든다
    - 복사본을 변경한다
    - 복사본을 리턴한다

**방어적 복사**

- 신뢰할 수 없는 코드와 데이터를 주고 받아야 할 때
- 안전지대의 경계에서 데이터가 오고 갈 때 방어적 복사를 사용
- 깊은 복사
- 규칙
    - 안전지대로 들어오는 데이터에 깊은 복사를 만든다.
    - 안전지대에서 나가는 데이터에 깊은 복사를 만든다.

<br/>
<br/>


## 자바스크립트에서 깊은 복사를 구현하는 것은 어렵다

- 자바스크립트는 표준 라이브러리가 좋지 않아 깊은 복사를 제대로 구현하는 것이 어렵다
- Lodash 라이브러리의 `.cloneDeep()` 함수를 사용하는 것을 추천
- `.cloneDeep()` 는 중첩된 데이터에 깊은 복사를 한다.

<br/>
<br/>

## 요점 정리

- 방어적 복사는 불변성을 구현하는 원칙이다. 데이터가 들어오고 나갈때 복사본을 만든다.
- 방어적 복사는 깊은 복사를 한다 → 비용이 더 든다.
- 방어적 복사는 불변성 원칙을 구현하지 않은코드로부터 데이터를 보호한다.
- 방어적 복사는 신뢰할 수 없는 코드와 함께 사용할 때만 사용한다.
- 깊은 복사는 위에서 아래로 중첩된 데이터 전체를 복사한다.
- 얕은 복사는 필요한 부분만 최소한으로 복사한다.
