# Chapter4. 액션에서 계산 빼내기

1. 절차적인 방법으로 구현하기
2. 테스트하기 쉽게 만들기
    - DOM 업데이트와 비즈니스 규칙을 분리하기
    - 전역변수 제거하기
3. 재사용하기 쉽게 만들기
    - 전역변수 제거하기
    - DOM 업데이트와 비즈니스 규칙을 분리하기
    - 함수에서 결괏값을 리턴시키기
    

<aside>
💡 테스트, 재사용성, 함수의 입출력

</aside>

## 입력

함수가 계산을 하기 위한 외부 정보

## 출력

함수 밖으로 나오는 정보나 어떤 동작

- 입력과 출력은 명시적이거나 암묵적일 수 있다.
    - 명시적인 입력 : 인자
    - 명시적인 출력 : 리턴값
    - 암묵적 입력 : 인자 외 다른 입력(전역변수 읽기)
    - 암묵적 출력 : 리턴값 외 다른 출력(콘솔창 출력, 전역변수 재할당)
- 함수에 암묵적 입력과 출력이 있으면 액션이 된다.

> 함수형 프로그래머는 암묵적 입력과 출력을 **부수 효과** (Side Effect)라고 부른다. 부수 효과는 함수가 하려고 하는 주요 기능(리턴값을 계산하는 일)이 아니다.
> 

## Refactoring

- Before
    
    ```jsx
    var shopping_cart = [];
    var shopping_cart_total = 0;
    
    function update_shipping_icons() {
    	var buy_buttons = get_buy_buttons_dom();
    	for(var i = 0; i < buy_buttons.length; i++) {
    		var button = buy_buttons[i];
    		var item = button.item;
    		if(item.price + shopping_cart_total >= 20) {
    			button.show_free_shipping_icon();
    		} else {
    			button.hide_free_shipping_icon();
    		}
    	}
    }
    
    function update_tax_dom() {
    	set_tax_dom(shopping_cart_total * 0.10);
    }
    
    function calc_cart_total() {
    	shopping_cart_total = 0;
    	for(var i = 0; i < shopping_cart.length; i++) {
    		var item = shopping_cart[i];
    		shopping_cart_total += item.price;
    	}
    	set_cart_total_dom();
    	update_shipping_icons()
    }
    
    function add_item_to_cart(name, price) {
    	shopping_cart.push({
    		name: name,
    		price: price
    	});
    	calc_cart_Total();
    }
    ```
    
1. 액션에서 계산 빼내기
    - Code - Extract subroutine
        
        ```jsx
        // Before
        function calc_cart_total() {
        	shopping_cart_total = 0;
        	for(var i = 0; i < shopping_cart.length; i++) {
        		var item = shopping_cart[i];
        		shopping_cart_total += item.price;
        	}
        }
        
        	set_cart_total_dom();
        	update_shipping_icons();
        	update_tax_dom();
        }
        
        // After
        function calc_cart_total() {
        	calc_total();
        	set_cart_total_dom();
        	update_shipping_icons();
        	update_tax_dom();
        }
        
        function calc_total() {
        	shopping_cart_total = 0;
        	for(var i = 0; i < shopping_cart.length; i++) {
        		var item = shopping_cart[i];
        		shopping_cart_total += item.price;
        	}
        }
        ```
        
2. 액션을 계산으로 바꾸기 - 암묵적 입/출력 제거하기
    - Code
        
        ```jsx
        // Before
        function calc_cart_total() {
        	calc_total();
        	set_cart_total_dom();
        	update_shipping_icons();
        	update_tax_dom();
        }
        
        function calc_total() {
        	shopping_cart_total = 0;
        	// 전역변수 사용 - 암묵적 입력
        	for(var i= 0; i < shopping_cart.length; i++) {
        		var item = shopping_cart[i];
        		// 전역변수 사용 - 암묵적 출력
        		shopping_cart_total += item.price;
        	}
        }
        
        // After
        function calc_cart_total() {
        	shopping_cart_total = calc_total(shopping_cart);
        	set_cart_total_dom();
        	update_shipping_icons();
        	update_tax_dom();
        }
        
        function calc_total(cart) {
        	var total = 0;
        	for (var i = 0; i < cart.length;; i++) {
        		var item = cart(i);
        		total += item.price;
        	}
        }
        
        ```
        
3. Copy-on-write
    
    값을 복사해서 바꾸는 방법으로, **불변성**을 구현하는 방법 중 하나이다.
    
    - Code
        
        ```jsx
        // Before
        function add_item_to_cart(name, price) {
        	shopping_cart.push({
        		name: name,
        		price: price
        	});
        
        	calc_cart_total();
        }
        
        // After
        function add_item_to_cart(name, price) {
        	shopping_cart = 
        		add_item(shopping_cart, name, price);
        	alc_cart_total();
        }
        
        function add_item(cart, name, price) {
        	var new_cart = cart.slice();
        	new_cart.push({
        		naem: name,
        		price: price
        	});
        	return new_cart;
        }
        ```
        
    - 복사 비용
        
        배열을 바꾸는 것보다 배열을 복사하는 것이 비용이 더 드는 것은 맞다. 하지나 최신 프로그래밍 언어의 런타임과 가비지 컬렉터(garbage collector)는 불필요한 메모리를 효율적으로 처리한다.
        
- After
    
    ```jsx
    // 전역변수 = 액션
    var shopping_cart = [];
    var shopping_cart_total = 0;
    
    function add_item_to_cart(name, price) {
    	// 전역변수 읽기 = 액션
    	shopping_cart = add_item(shopping_cart, name, price);
    	calc_cart_total();
    }
    
    function calc_cart_total() {
    	shopping_cart_total = calc_total(shopping_cart);
    	set_cart_total_dom();
    	update_shipping_icons();
    	update_tax_dom();
    }
    
    function update_shipping_icons() {
    	var buttons = get_buy_buttons_dom();
    	for (var i = 0; i < buttons.length; i++) {
    		var button = buttons[i];
    		var item = button.item;
    		if (gets_free_shipping(shopping_cart_total, item.price)){
    			button.show_Free_shipping_icon();
    		} else {
    			button.hide_free_shipping_icon();
    		}
    	}
    }
    
    function update_tax_dom() {
    	set_tax_dom(calc_tax(shopping_cart_total));
    }
    
    function add_item(cart, name, price) {
    	var new_cart = cart.slice();
    	new_cart.push({
    		name: name,
    		price: price
    	});
    	return new_cart;
    }
    
    function calc_total(cart) {
    	var total = 0;
    	for(var i = 0; i < cart.length; i++) {
    		var item = cart[i];
    		total += item.price;
    	}
    	return total;
    }
    
    function gets_free_shipping(total, item_price) {
    	return item_price + total >= 20;
    }
    
    function clac_tax(amount) {
    	return amount * 0.10;
    }
    ```
    

## 예제 풀이

- 1
    
    ```jsx
    // 세금 계산 부분 추출
    function update_tax_dom() {
    	set_tax_dom(shopping_cart_total * 0.10);
    }
    
    // 입력은 인자로 바꾸고 출력은 리턴값으로 바꾼다.
    ```
    
    ```jsx
    function calculate_tax(value) {
    	return value * 0.10;
    }
    
    function update_tax_dom(){
    	set_tax_dom(calculate_tax(shopping_cart_total));
    }
    ```
    
- 2
    
    ```jsx
    // 무료 배송인지 확인하는 코드 추출하기
    // update_shipping_icons() 함수에서 계산 추출하기
    // Before
    function update_shipping_icons() {
    	var buy_buttons = get_buy_buttons_dom();
    	for (var i = 0; i < buy_buttons.length; i++) {
    		var button = buy_buttons[i];
    		var item = button.item;
    		if(item.price + shopping_cart_total >= 20){
    			button.show_free_shipping_icon();
    		} else {
    			button.hide_free_shipping_icon();
    		}
    	}		
    }
    ```
    
    ```jsx
    function check_free_shipping(price, total) {
    	return price + total >= 20;
    }
    
    function update_shipping_icons() {
    	var buy_buttons = get_buy_buttons_dom();
    	for (var i = 0; i < buy_buttons.length; i++) {
    		var button = buy_buttons[i];
    		var item = button.item;
    		if(check_free_shipping(item.price, shopping_cart_total) {
    			button.show_free_shipping_icon();
    		} else {
    			button.hide_free_shipping_icon();
    		}
    	}
    }
    ```
    

## Summary

1. 액션은 암묵적인 입력 또는 출력을 가지고 있다.
2. 계산은 암묵적인 입력이나 출력이 없어야 한다.
3. 전역 변수는 암묵적 입력 또는 출력이 된다.
4. 암묵적 입력은 인자로 바꿀 수 있다.
5. 암묵적 출력은 리턴값으로 바꿀 수 있다

# Chapter 5. 더 좋은 액션 만들기

1. 비즈니스 요구 사항과 설계를 맞추기
    
    요구 사항에 맞춰 더 나은 추상화 단계 선택하기
    
    - e.g.
        
        ```jsx
        // 요구 사항 : 장바구니에 담긴 제품을 주문할 때 무료 배송인지 확인하는 것
        // Before
        function gets_free_shipping(total, item_price) {
        	return item_price + total >= 20;
        }
        
        function calc_total(cart) {
        	var total = 0;
        	for(var i = 0; i < cart.length; i++) {
        		var item = cart[i];
        		// 합계를 계산하는 코드 중복
        		total += item.price;
        	}
        	return total;
        }
        
        //After
        function gets_free_shipping(cart){
        	return calc_total(cart) >= 20;
        }
        
        function update_shipping_icons() {
        	var buttons = get_buy_buttons_dom();
        	for(var i = 0; i < buttons.length; i++) {
        		var button = buttons[i];
        		var item = button.item;
        		var new_cart = add_item(shopping_cart, item.name, item.price);
        		if(gets_free_shipping(new_Cart)) {
        			button.show_free_shipping_icon();
        		} else {
        			button.hide_free_shipping_icon();
        		}
        	}
        }
        
        ```
        
2. 암묵적 입력과 출력 줄이기
    
    암묵적 입력과 출력이 있다면 다른 컴포넌트와 강하게 연결된 컴포넌트라고 할 수 있다. 이런 함수의 동작은 연결된 부분의 동작에 의존한다.
    
    암묵적 입력과 출력이 있는 함수는 아무 때나 실행할 수 없기 때문에 테스트하기 어렵다. 모든 입력값을 설정하고 테스트를 돌린 후에 모든 출력값을 확인해야 하기 때문이다.
    
    - Code
        
        ```jsx
        // 암묵적 입력을 명시적 입력인 인자로 바꾸기
        // Before
        function update_shipping_icons() {
        	var buttons = get_buy_buttons_dom();
        	for(var i = 0; i < buttons.length; i++) {
        		var button = buttons[i];
        		var item = button.item;
        		var new_cart = add_item(shopping_cart, item.name, item.price);
        		if(gets_Free_shipping(new_cart)){
        			button.show_free_shipping_icon();
        		} else {
        			button.hide_free_shipping_icon();
        		}
        	}
        }
        
        function calc_cart_total() {
        	shopping_cart_total = calc_total(shopping_cart);
        	set_cart_total_dom();
        	update_shipping_icons();
        	update_tax_dom();
        }
        
        // After
        function update_shipping_icons(cart) {
        	var buttons = get_buy_buttons_dom();
        	for(var i = 0; i < buttons.length; i++) {
        		var button = buttons[i];
        		var item = button.item;
        		var new_cart = add_item(cart, item.name, item.price);
        		if (gets_free_shipping(new_cart)) {
        			button.show_free_shipping_icon();
        		} else {
        			button.hide_free_shipping_icon();
        		}
        	}
        }
        
        function calc_cart_total() {
        	shopping_cart_total = calc_total(shopping_cart);
        	set_cart_total_dom();
        	update_shipping_icons(shopping_cart);
        	update_tax_dom();
        }
        ```
        
3. 계산 분류하기
    
    후에 계층으로 분리 할 수 있음
    
4. 유틸리티 함수 만들기
    - e.g.
        
        ```jsx
        // Before
        function add_item(cart, name, price) {
        	var new_cart cart.slice();
        	new_cart.push({
        		name: name,
        		price: price
        	});
        	return new_cart;
        }
        
        // After
        function add_element_last(array, elem) {
        	var new_array = array.slice();
        	new_array.push(elem);
        	return new_array;
        }
        ```
        

## 예제 풀이

- Code
    
    ```jsx
    function update_shipping_icons() {
    	var buy_buttons = get_buy_buttons_dom();
    	for(var i = 0; i < buy_buttons.length; i++) {
    		var button = buy_buttons[i];
    		var item = button.item;
    		var new_cart = add_item(cart, item);
    		if(gets_Free_shipping(new_cart)){
    			button.show_free_shipping_icon();
    		} else {
    			button.hide_free_shipping_icon();
    		}
    	}
    }
    ```
    
    ```jsx
    function check_is_free_shipping(cart, item) {
    	return gets_free_shipping(add_item(cart, item));
    }
    
    function handle_free_shipping_icon(button, isShown) {
    	if(isShown) {
    		button.show_free_shipping_icon();
    	} else {
    		button.hide_free_shipping_icon();
    	}
    }
    
    function update_shipping_icons(cart) {
    	var buttons = get_buy_buttons_dom();
    	for (var i = 0; i < buttons.length; i++) {
    		var button = buttons[i];
    		handle_free_shipping_icon(button, check_is_free_shipping(cart, button.item));
    	}
    }
    
    ```
    

## Summary

1. 암묵적 입력과 출력은 인자와 리턴값으로 명시적으로 바꾸는 것이 좋다.
2. 설계는 엉켜있는 것을 푸는 것이다.
3. 하나의 함수가 하나의 일을 하도록 하면, 쉽게 구성할 수 있다.
