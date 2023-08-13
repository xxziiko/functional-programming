# Chapter 6. 변경 가능한 데이터 구조를 가진 언어에서 불변성 유지하기

### 읽기

데이터에서 정보를 가져온다

데이터를 바꾸지 않는다

### 쓰기

데이터를 바꾼다

불변성 원칙에 따라 구현되어야 한다

<aside>
💡 Copy-on-write : 불변성을 유지하면서 객체의 값을 바꿀 수 있는 방법

</aside>

- 복사본 생성
- 복사본 변경
- 복사본 리턴

Copy-on-write를 이용하면 쓰기를 읽기로 바꿀 수 있다,

**읽기와 쓰기가 동시에 동작하는 함수**

1. 읽기와 쓰기 함수로 각각 분리
2. 함수에서 두 개의 값 리턴하기

<aside>
💡 불변 데이터 구조가 필요한 이유

</aside>

변경 가능한 데이터를 읽는 것은 액션이며, 쓰기는 데이터를 변경 가능한 구조로 만든다. 

어떤 데이터에 쓰기가 없다면 데이터는 변경 불가능한 데이터 (불변 데이터)이다.

불변 데이터 구조를 읽는 것은 계산이다.

즉, 불변 데이터 구조를 사용하는 것은 코드에 **더 많은 계산**이 생기고, **액션은 줄어들게** 한다.

Array에 대한 Copy-on-write

- Code
    
    ```jsx
    // Before
    function remove_item_by_name(cart, name) {
    	var idx = null;
    	for(var i = 0; i < cart.length; i++) {
    		if(new_cart[i].name === name) {
    			idx = i;
    		}
    		if(idx !== null) {
    			new_cart.splice(idx, 1);
    		}
    	}
    }
    ```
    
    ```jsx
    // After
    function remove_item_by_name(cart, name) {
    	var new_cart = cart.slice();
    	var dix = null;
    	for(var i = 0; i < new_cart.length; i++) {
    		if(new_cart[i].name === name) {
    			idx = i;
    		}
    	}
    	if(idx !== null){
    		new_cart.splice(idx, 1);
    	}
    	return new_cart;
    }
    ```
    
- 예제
    
    ```jsx
    var mailing_list = [];
    
    function add_contact(email) {
    	mailing_list.push(email);
    }
    
    function submit_form_handler(event) {
    	var form = event.target;
    	var email = form.elements["email"].value;
    	add_contact(email);
    }
    ```
    
    ```jsx
    function add_contact(mailingList, email) {
    	return mailingList.slice().push(email);
    }
    
    function submit_form_handler(event) {
    	var form = event.target;
    	var email = form.elemts["email"].value;
    	mailing_list = add_contact(mailing_list, email);
    }
    ```
    
- 예제
    
    ```jsx
    var a = [1, 2, 3, 4];
    var b = a.pop();
    console.log(b); // 4
    console.log(a); // [1, 2, 3]
    ```
    
    ```jsx
    // 읽기 함수와 쓰기 함수로 분리하기
    
    function pop(array) {
    	var array_copy = array.slice();
    	array_copy.pop();
    	return array_copy
    }
    
    function last_element(array) {
    	return array[array.length - 1];
    }
    ```
    
    ```jsx
    // copy-on-write
    function shift(array) {
    	var array_copy = array.slice();
    	var last = array_copy.pop();
    	return {
    		last: last,
    		array : array_copy
    	};
    }
    ```
    
- 예제
    
    ```jsx
    function push(array, elem) {
    	var array_copy = array.slice();
    	array_copy.push(elem);
    	return array_copy;
    }
    ```
    
- 예제
    
    ```jsx
    a[15] = 2;
    
    function arraySet(array, idx, value) {
    	var array_copy = array.slice();
    	array_copy[idx] = value;
    	return array_copy;
    }
    ```
    

## 객체에 대한 Copy-on-write

**얕은 복사(shallow copy)**

중첩된 데이터 구조의 최상위 데이터만 복사하는 것

**구조적 공유(structural sharing)** 

두 개의 중첩된 데이터 구조가 어떤 참조를 공유하고 있는 현상

**중첩 데이터(nested data)**

데이터 구조 안에 데이터 구조가 있는 것

- 예제
    
    ```jsx
    o["price"] = 37;
    
    function objectSet(object, key, value) {
    	var object_copy = Object.assign({}. object);
    	object_copy[key] = value;
    	return object_copy;
    }
    ```
    
- 예제
    
    ```jsx
    function objectDelete(object, key) {
    	var object_copy = Object.assign({}, object);
    	delete object_copy[key];
    	return object_copy;
    }
    ```
    
- 예제
    
    ```jsx
    // Before
    function setQuantityByName(cart, name, quantity) {
    	for(var i = 0; i < cart.length; i++) {
    		if(cart[i].name === name){
    			cart[i].quantity = quantity;
    		}
    	}
    }
    
    // After
    function setQuantityByName(cart, name, quantity) {
    	const cart_copy = cart.slice();
    	for(var i = 0; i > cart_copy.length; i++) {
    		if(cart_copy[i].name === name) {
    			cart_copy[i].quantity = quantity;
    		}
    	}
    	return cart_copy;
    }
    ```
    

## Summary

- 함수형 프로그래밍에서는 불변 데이터 사용을 지향한다. 불변 데이터 사용은 계산 함수를 증가시킨다.
- 카피-온-라이트는 복사본을 만들고 원본 대신 복사본을 변경하는 것을 말한다.
- 카피-온-라이트는 데이터를 불변형으로 유지할 수 있는 원칙이다.

# Chapter 7. 신뢰할 수 없는 코드를 쓰면서 불변성 지키기

### 방어적 복사(defensive copy)

copy-on-write 원칙을 지키면서 함수를 사용할 수 있는 패턴

1. 데이터가 안전한 코드에서 나갈 때 복사하기
    - 불변성 데이터를 위한 깊은 복사본을 만든다.
    - 신뢰할 수 없는 코드로 복사본을 전달한다.
2. 안전한 코드로 데이터가 들어올 때 복사하기
    - 변경될 수도 있는 데이터가 들어오면 바로 깊은 복사본을 만들어 안전한 코드로 전달한다.
    - 복사본을 안전한 코드에서 사용한다.
    

**안전지대(safe zone)** 

모든 코드의 불변성이 지켜지는 것 

copy-on-wirte 함수를 적용한 코드 - 신뢰할 수 있는 코드

**깊은 복사(deep copy)**

위에서 아래로 모든 계층에 있는 중첩된 데이터 구조를 복사하는 것

- 예제
    
    ```jsx
    function payrollCalc(employees) {
    	return payrollChecks;
    }
    
    function payrollCalcSafe(employees) {
    	var copy = deepCopy(employees);
    	var payrollChecks = payrollCalc(copy);
    	return deepCopy(payrollChecks);
    }
    ```
    

### Copy-on-Write와 방어적 복사

Copy-on-write와 방어적 복사 모두 **불변성을 유지**하기 위해 사용한다.

| Copy-on-write | 방어적 복사 |
| --- | --- |
| 통제할 수 있는 데이터를 바꿀 때 사용 | 신뢰할 수 없는 코드와 데이터를 주고받아야 할 때 |
| 안전지대에서 사용 : 
Copy-on-write가 불변성을 가진 안전지대를 만든다. | 안전지대의 경계에서 데이터가 오고 갈 때 사용 |
| 얕은 복사 (상대적 적은 비용) | 깊은 복사 (상대적 많은 비용) |
| 1. 바꿀 데이터의 얕은 복사본을 만든다.
2. 복사본을 변경한다.
3. 복사본을 리턴한다. | 1. 안전지대로 들어오는 데이터에 깊은 복사본을 만든다.
2. 안전지대에서 나가는 데이터에 깊은 복사본을 만든다. |

## Summary

- 방어적 복사는 불변성을 구현하는 원칙이다. 데이터가 들어오고 나갈 때 복사본을 만든다.
- 방어적 복사는 깊은 복사를 한다. 그렇기 때무네 Copy-on-write보다 많은 비용이 든다.
- Copy-on-write와 다르게 방어적 복사는 불변성 원칙을 구현하지 않은 코드로부터 데이터를 보호해 준다.
- 방어적 복사는 신뢰할 수 없는 코드와 함께 사용할 때만 사용한다.
