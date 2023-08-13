# Chapter 8 - 9. 계층형 설계

**소프트웨어 설계** (software design)

코드를 만들고, 테스트하고, 유지보수하기 쉬운 프로그래밍 방법을 선택하기 위한 단계

**계층형 설계** (stratified designd)

소프트웨어를 계층으로 구성하는 기술

각 계층에 있는 함수는 바로 아래 계층에 있는 함수를 이용해 정의한다. 

특정 구체화 단계에 집중할 수 있게 도와준다.

# 계층형 설계 패턴

## 직접 구현

직접 구현은 계층형 설계 구조를 만드는데 도움이 된다. 

직접 구현된 함수를 읽을 때, 함수 시그니처가 나타내고 있는 문제를 함수 본문에서 적절한 구체화 수준에서 해결해야 한다.

- 예시
    
    ```jsx
    function freeTieClip(cart) {
    	var hasTie = false;
    	var hasTieClip = false;
    	for(var i = 0; i < cart.length; i++) {
    		var item = cart[i];
    		if(item.name === 'tie') {
    			hasTie = true;
    		}
    		if(item.name === 'tie clip') {
    			hasTieClip = true;
    		}
    	}
    	if(hasTie && !hasTieClip){
    		var tieClip = make_item("tie clip", 0);
    		return add_item(cart, tieClip);
    	}
    	return cart;
    }
    ```
    
    직접 구현을 따르지 않고 있음 - 알 필요가 없는 구체적인 내용을 담고 있다.
    
    마케팅 캠페인에 관련된 함수가 장바구니가 배열이라는 사실을 알아야 할까?
    
    ```jsx
    // 장바구니 안에 제품이 있는지 확인하는 함수
    function isInCart(cart, name) {
    	for(var i = 0; i < cart.length; i++) {
    		if(cart[i].name === name){
    			return true;
    		}
    		return false;
    	}
    }
    ```
    

## 추상화 벽

세부 구현을 감춘 함수로 이루어진 계층이다.

인터페이스를 사용하여 코드를 만들면 높은 차원으로 생각할 수 있다.

벽을 마주보고 일하는 사람 간 책임을 명확하게 나눌 수 있다. 

라이브러리 혹은 API

- 예제
    
    ```jsx
    // Before : 배열로 만든 장바구니
    function add_item(cart, item) {
    	return add_element_last(cart, item);
    }
    
    function calc_total(cart) {
    	var total = 0;
    	
    	for(var i = 0; i < cart.length; i ++) {
    		var item = cart[i];
    		total += item.price;
    	}
    	return total;
    }
    
    function setPriceByName(cart, name, price) {
    	var cartCopy = cart.slice();
    	for(var i = 0; i < cartCopy.length; i++) {
    		if(cartCopy[i].name === name) {
    			artCopy[i] = setPrice(cartCopy[i].price);
    		}
    	}
    	return cartCopy;
    }
    
    function remove_item_by_name(cart, name) {
    	var idx = indexOfItem(cart, name);
    	if(idx !== null) {
    		return spice(cart, idx, 1);
    	}
    	return cart;
    }
    
    function indexOfItem(cart, name) {
    	for(var i = 0; i < cart.length; i++) {
    		if(cart[i].name === name) {
    			return i;
    		}
    	}
    	return null;
    }
    
    function isInCart(cart, name) {
    	return indexOfItem(cart, name) !== null;
    }
    ```
    
    ```jsx
    // After: 객체로 만든 장바구니
    function add_item(cart, name) {
    	return ObjectSet(cart, item.name, item);
    }
    
    function calc_total(cart) {
    	var total = 0;
    	var names = Object.keys(cart);
    	for(var i = 0; i < names.legth; i++) {
    		var item = cart[names[i]];
    		total += item.price;
    	}
    	return total;
    }
    
    function setPriceByName(cart, name, price){
    	if(isInCart(cart, name)){
    		var item = cart[name];
    		var copy = setPrice(item, price);
    		return objectSet(cart, name, copy);
    	} else {
    		var item = make_item(name, price);
    		return objectSet(cart, name, item);
    	}
    }
    
    function remove_item_by_name(cart, name) {
    	return objectDelete(cart, name);
    }
    
    function isInCart(cart, mame) {
    	return cart.hasOwnProperty(name);
    }
    ```
    

추상화 벽과 그 아래에 있는 코드는 높은 수준의 계층에서 함수가 어떻게 사용되는지 몰라도 된다 

추상화 단계의 상위에 있는 코드와 하위에 있는 코드는 서로 의존하지 않게 정의한다.

추상화 단계의 모든 함수는 비슷한 세부 사항을 무시할 수 있도록 정의한다.

추상화 벽은 팀 간 커뮤니케이션 비용을 줄이고, 복잡한 코드를 명확하게 하기 위해 전략적으로 사용해야 한다.

추상화 벽으로 추상화 벽 아래에 있는 코드와 위에 있는 코드의 의존성을 없앨 수 있다.

## 작은 인터페이스

새로운 기능을 추가할 때 하위 계층에 기능을 추가하거나 고치는 것보다 상위 계층에서 문제를 해결할 수 있게 만드는 패턴

인터페이스를 최소화 하면 하위 계층에 기능이 커지는 것을 막을 수 있다.

- 예시
    
    장바구니에 제품을 100달러 이상 담은 사람이 시계를 구입하면 10% 할인
    
    방법 1 : 추상화 벽에 만들기
    
    추상화 벽 계층에 있으면 해시 맵 데이터 구조로 되어 있는 장바구니에 접근할 수 있다. 하지만 같은 계층에 있는 함수를 사용할 수 없다.
    
    ```jsx
    function getWatchdiscount(cart) {
    	var total = 0;
    	var names = Object.keys(cart);
    	for(var i = 0; i < names.length; i++) {
    		var item = cart[names[i]];
    		total += item.price;
    	}
    	return total > 100 && cart.hasOwnProperty("watch");
    };
    ```
    
    방법 2 : 추상화 벽에 만들기
    
    추상화 벽 위에 만들면 해시 데이터 구조를 직접 접근할 수 없다. 추상화 벽에 있는 함수를 사용하여 장바구니에 접근해야 한다.
    
    ```jsx
    function getWatchDiscount(cart) {
    	var total = calcTotal(cart);
    	var hasWatch = isinCart("watch");
    	return total > 100 && hasWatch;
    }
    ```
    

## 편리한 계층

계층형 설계 패턴과 실천 방법은 개발자의 요구를 만족 시키면서 비즈니스 문제를 잘 풀 수 있어야 한다. 편리한 계층 패턴은 언제 패턴을 적용하고 또 언제 멈춰야 하는지 실용적인 방법을 알려준다.   

# 호출 그래프

함수 호출을 시각화 할 수 있는 방법 

- 한 함수에서 서로 다른 **추상화** 단계를 사용하면 코드가 명확하지 않아 읽기 어려울 수 있기 때문에, 함수가 모두 비슷한 계층에 있을 수 있도록 구현
- 같은 계층에 있는 함수는 같은 목적을 가져야 한다.
- 각 계층의 추상화 수준이 다르기 때문에, 어떤 계층에 있는 함수를 읽거나 고칠 때 낮은 수준의 내용은 신경쓰지 않아도 된다.
- 호출 그래프의 줌 레벨
    - 전역 줌 레벨 - 기본 줌 레벨
        
        계층 사이의 상호 관계를 포함해서 모든 문제 영역을 살펴 볼 수 있다.
        
    - 계층 줌 레벨
        
        한 계층과 연결된 바로 아래 계층을 볼 수 있는 줌 레벨
        
    - 함수 줌 레벨
        
        함수 하나와 바로 아래 연결된 함수들을 볼 수 있다.
        
- 호출 그래프의 화살표 길이
    
    직접 구현 패턴을 사용하면 모든 화살표가 같은 길이를 가져야 한다. 다양한 화살표 길이를 가지고 있다는 것은 다양한 계층을 넘나드는 뜻으로 같은 구체화 수준이 아니라는 증거이다.
    
- 예시

- 호출 그래프로 알 수 있는 세 가지 비기능적 요구사항
    1. 유지보수성(maintainability) : 요구 사항이 바뀌었을 때 가장 쉽게 고칠 수 있는 코드는 어떤 코드일까?
        
        위로 연결된 것이 적은 함수가 바꾸기 쉬우며, 자주 바뀌는 코드는 가능한 위쪽에 있어야 한다.
        
    2. 테스트성(testability) : 어떤 것을 테스트하는 것이 가장 중요할까?
        
        위쪽으로 많이 연결된 함수(아래쪽에 있는 함수)를 테스트하는 것이 더 가치 있다.
        
    3. 재사용성(reusability) : 어떤 함수가 재사용하기 좋을까?
        
        아래쪽에 함수가 적을 수록 (낮은 수준의 함수일수록) 재사용성이 높아진다.
        

# Summary

- 계층형 설계는 바로 아래 계층에 있는 함수로 현재 계층의 함수를 구현하는 기술이다.
- 계층형 설계는 코드를 추상화 계층으로 구성한다. 추상화된 각 계층을 볼 때는 다른 계층의 구체적인 내용을 알지 못해도 된다.
- 문제 해결을 위한 함수를 구현할 때는 어떤 구체화 단계로 구현할지 결정하는 것이 중요하다.
- 구체화 단계를 정하는 요소 (어떤 계층에 속할지 알려주는 요소) - 함수 이름, 본문, 호출 그래프 등
    - 함수 이름은 의도를 알려준다. 비슷한 목적의 이름을 가진 함수는 함께 묶을 수 있다.
    - 함수 본문은 중요한 세부 사항을 알려준다. 함수 본문은 함수가 어떤 계층 구조에 있어야 하는지 알려준다.
    - 호출 그래프로 구현이 직접적이지 않다는 것을 알 수 있다. 함수를 호출하는 화살표가 다양한 길이를 가지고 있다면 직접 구현되어 있지 않다는 신호이다.

- 직접 구현 패턴은 함수를 명확하고 아름답게 구현해 계층을 구성할 수 있도록 알려준다.
- 추상화 벽 패턴을 사용하면 세부적인구조를 감출 수 있기 때문에 더 높은 차원에서 생각할 수 있다.
- 작은 인터페이스 패턴을 사용하면 완성된 인터페이스에 가깝게 계층을 만들 수 있다.
- 편리한 계층 패턴을 사용하면 다른 패턴을 요구 사항에 맞게 사용할 수 있다.
- 호출 그래프 구조에서는 규칙을 쉽게 파악할 수 있다.
