

# Chapter 12. 함수형 반복

## Array.prototype.map( )

```jsx
arr.map(callback(currentValue[, index[, array]])[, thisArg])
```

`map`은 `callback` 함수를 각각의 요소에 대해 한번씩 순서대로 불러 그 함수의 반환값으로 새로운 배열을 만든다. `callback` 함수는 배열 값이 들어있는 인덱스에 대해서만 호출된다.

map은 호출한 배열의 값을 변형하지 않지만, callback 함수에 의해서 변형될 수는 있다.

- 예시
    
    ```jsx
    // Before
    function emailsForCustomers(customers, goods, bests) {
    	var emails = [];
    	forEach(customers, function(customer) {
    		var email = emailForCustomer(customer, goods, bests);
    		emails.push(email);
    	});
    	return emails;
    }
    
    // After
    function emailsForCustomers(customers, goods, bests) {
    	return map(customers, function(customer) {
    		return emailForCustomer(customer, goods, bests);
    	};
    }
    
    function map(array, f) {
    	var newArray = [];
    	forEach(array, function(element) {
    		newArray.push(f(element));		
    	});
    	return newArray;
    }
    ```
    

## Array.prototype.filter( )

```jsx
arr.filter(callback(element[, index[, array]])[, thisArg])
```

`filter()`는  배열 내 각 요소에 대해 한 번 제공된 `callback` 함수를 호출해, `callback`이 true로 반환하는 값이 있는 새로운 배열을 생성한다. `callback`은 할당된 값이 있는 배열의 인덱스에 대해서만 호출된다. 삭제됐거나 값이 할당된 적이 없는 인덱스에 대해서는 호출되지 않는다.

- 예시
    
    ```jsx
    // Before
    function selectBestCustomers(customers) {
    	var newArray = [];
    	forEach(customers, function(customer) {
    		if(customer.purchases.length >= 3) {
    			newArray.push/9customer);
    		}
    	});
    	return newArray;
    }
    
    // After
    function selectBestCustomers(customers) {
    	return filter(customers, function(customer) {
    		return customer.purchases.length >= 3;
    	});
    }
    
    function filter(array, f) {
    	var newArray = [];
    	forEach(array, function(element) {
    		if(f(element)){
    			newArraypush(element);
    		}
    	});
    	return newArray;
    }
    ```
    

## Array.prototype.reduce( )

```jsx
arr.reduce(callback[, initialValue])
```

`reduce()`메서드는 배열의 각 요소에 대해 주어진 `reducer`함수를 실행하고, 하나의 결과값을 반환한다.

callback 함수는 `accumulator, currentValue, currentIndex, array` 네 가지의 인수를 받는다.

initialValue를 제공하지 않으면, reduce()는 인덱스 1부터 시작해 콜백 함수를 

- 예시
    
    ```jsx
    // Before
    function countAllPurchases(customers) {
    	var total = 0;
    	forEach(customers, function(customer) {
    		total = total + customer.purchases.length;
    	});
    	return total;
    }
    
    // After
    function countAllPurchases(customers) {
    	return reduce (
    		customers, 0, function(total, customer) {
    			return total + customer.purchases.length;
    		}
    	);
    }
    
    function reduce(array, init, f) {
    	var accum = init;
    	forEach(array, function(element) {
    		accum = f(accum, element);
    	});
    	return accum;
    }
    ```
    

# Chapter 13. 함수형 도구 체이닝

- 예시
    
    ```jsx
    function maxKey(array, init, f) {
    	return reduce(array, 
    								init,
    								function(biggestSoFra, element) {
    									if(f(biggestSoFar) > f(element)) return biggestSoFar;
    										else return element;
    								});
    }
    ```
    
    ```jsx
    function biggestPurchasesBestCustomers(customers) {
    	var bestCUstomers = filter(customers, function(customer) {
    		return customer.purchases.length >= 3;
    	};
    	
    	var biggestPurchases = map(bestCustomers, function(customer) {
    		return maxKey(customer.purchases, {total: 0}, function(purchase) {
    			return purchase.total;
    		});
    	};
    }
    ```
    

항등 함수(identity function) - 인자로 받은 값을 그대로 리턴하는 함수

- 예시
    
    ```jsx
    function max(array, init) {
    	return maxKey(array, init, function(x) {
    		return x;
    	});
    }
    ```
    

체인을 명확하게 만들기

- 예시
    
    ```jsx
    // 방법 1: 단계에 이름 붙이기
    function biggestPurchasesBestCustomers(customes) {
    	var bestCustomers = selectBestCustomers(customers);
    	var biggestPurchases = getBiggerPurchasese(bestCustomers);
    	return biggestPurchasese;
    }
    
    function selectBestCustomers(customers) {
    	return filter(customers, function(customer) {
    		return customer.purchasese.length >= 3;
    	});
    }
    
    function getBiggestPurchasese(customers) {
    	return map(customers, getBiggestPurchase);
    }
    
    function getBiggestPurchase(customer) {
    	return maxKey(customer.purchasese, {total: 0}, function(purchase) {
    		return purchase.total;
    	});
    }
    ```
    
    ```jsx
    // 방법 2: 콜백에 이름 붙이기
    function biggestPurchaseseBestCustomers(customers) {
    	var bestCustomers = filter(customers, isGoodCustomer);
    	var biggestPUrchasese = map(bestCustomers, getBiggestPurchase);
    	return biggestPurchases;
    }
    
    function isGoodCustomer(customer) {
    	return customer.purchasese.length >= 3;
    }
    
    function getBiggestPUrchase(customer) {
    	return maxKey(customer.purchasese, {total: 0}, getPurchaseTotal);
    }
    
    function getPurchaseTotal(purchase) {
    	return purchase.total;
    }
    ```
    

**스트림 결합** (stream fusion)

## 반복문을 함수형 도구로 리팩토링하기

1. 데이터 만들기
- 예제
    
    ```jsx
    // Before
    var answer = [];
    var window = 5;
    
    for(var i=0; i < array.length; i++) {
    	var sum = 0;
    	var count = 0;
    	for(var w = 0; w < window; w++) {
    		var idx = i + w;
    		if(idx < array.length) {
    			sum += array[idx];
    			count += 1;
    		}
    	}
    	answer.push(sum/count);
    }
    
    // After
    var answer = [];
    var window = 5;
    
    for(var i=0; i < array.length; i++) {
    	var sum = 0;
    	var count = 0;
    	var subarray = array.slice(i, i + window);
    	for(var w = 0; w < window; w++) {
    		sum += subarray[w];
    		count += 1;
    	}
    	answer.push(sum/count);
    }
    ```
    
1. 한 번에 전체 배열을 조작하기
- 예제
    
    `1`에서 하위 배열을 만들었기 때문에 일부 배열이 아닌 배열 전체를 반복 할 수 있다.
    
    ```jsx
    // Before
    var answer = [];
    var window = 5;
    
    for(var i = 0; i < array.length; i++) {
    	var sum = 0;
    	var count = 0;
    	var subarray = array.slice(i, i+5);
    	for(var w = 0; w < subarray.length; w++) {
    		sum += subrray[w];
    		count += 1;
    	}
    	answer.push(sum/count);
    }
    
    // After
    var answer = [];
    var window = 5;
    
    for(var i = 0; i < array.length; i++) {
    	var subarray = array.slice(i, i + window);
    	answer.push(average(subarray));
    }
    ```
    
1. 작은 단계로 나누기

```jsx
// Before
var answer = [];
var window = 5;

for(var i = 0 ; i < array.length; i++) {
	var sum = 0;
	var count = 0;
	for(var w = 0; w < window; w++) {
		var idx = i + w;
		if(idx < array.length) {
			sum += array[idx];
			count += 1;
		}
	}
	answer.push(sum/count);
}

// After
var window = 5;

var indices = range(0, array.length);
var windows = map(indices, function(i) {
	return array.slice(i, i + window);
});
var answer = map(windows, average);

function range(start, end) {
	var ret = [];
	for(var i = start; i < end; i++) ret.push(i);
	return ret;
}
```

`pluck()`

```jsx
function pluck(array, field) {
	return map(array, function (object) {
		return object[field];
	});
}
```

`concat()`

중첩된 배열을 한 단계의 배열로 만든다.

```jsx
function concat(arrays) {
	var ret = [];
	forEach(arrays, function(array) {
		forEach(array, function(element) {
			ret.push(element);
		});
	});
	return ret;
}
```

`frequenciesBy()`, `groupBy()`

이 함수는 객체 또는 맵을 리턴한다.
