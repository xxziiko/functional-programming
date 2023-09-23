# chapter 12.

- `map()`, `filter()`, `reduce()` 로 반복문을 대체해서 코드의 목적을 더 명확하게 할 수 있다.

## 함수형 도구: `map()`

```jsx
function map (array, f) {
	const newArray = [];
	forEach(array, function (element) {
		newArray.push(f(element));
	
	});
}
```

- 배열의 모든 항목에 함수를 적용해 새로운 배열로 바꿈
- 각 항목은 지정한 콜백 함수에 의해 변환

## 함수형 도구: `filter()`

```jsx
function filter(array, f) {
	const newArray = [];
	forEach(array, function (element) {
		if(f(element)) return newArray.push(element);
	})

return newArray;
}
```

- 배열의 하위 집합을 선택해 새로운 배열로 만듬
- 술어를 전달해서 특정 항목을 선택할 수 있음
    - 술어 : `true` 또는 `false` 를 리턴하는 함수
    

## 함수형 도구: `reduce()`

```jsx
function reduce(array, init, f) {
	const arrum = init;
	forEach(array, function(element) {
		accum = f(accum, element);
	});
	return accum;
}
```

- 배열을 순회하면서 값을 누적
- 함수는 누적하고 있는 현재 값과 반복하고 있는 현재 배열의 항목을 인자로 받는다. 그리고 새로운 누적값을 리턴한다.
- 주의할점! 이 책에서는 인자 순서를 **배열, 초기값, 콜백 순**으로 정했다.

## `reduce()`로 할 수 있는 것들

- 실행취소/ 실행 복귀
- 테스트 할 때 사용자 입력을 다시 실행하기
- 시간여행 디버깅
    - 특정 시점 상태의 값 보관
- 회계 감사 추적
    - 과거에 어떤 일이 있었는지 기록
    

# chapter 13.

- 복잡한 반복문을 함수형 도구 체인으로 바꾸는 방법

## 체인을 명확하게 만들기 1: 단계에 이름 붙이기

```jsx
function biggestPurchasesBestCustomers (customers) {
	  const bestCustomers = filter(customers, function (customers) {
		return customers.purchases.length >= 3;
	});
		
		...
		return bestPurchases;
}

// 이름 붙이기 
function biggestPurchasesBestCustomers (customers) {
	  const bestCustomers = selectBestCustomers(customers);
		const bestPurchases = getBiggestPurchases(bestCustomers);
		return bestPurchases;
}

function selectBestCustomers (customers) {
	return filter(customers, function (customers) {
		return customers.purchases.length >= 3;
	});
}

...
```

## 체인을 명확하게 만들기 2: 콜백에 이름 붙이기

```jsx
function biggestPurchasesBestCustomers (customers) {
	  const bestCustomers = filter(customers, function (customers) {
		return customers.purchases.length >= 3;
	});
		
		...
		return bestPurchases;
}

function biggestPurchasesBestCustomers (customers) {
	  const bestCustomers = filter(customers, isGoodCustomer);
		
		...
		return bestPurchases;
}

function isGoodCustomer (customer) {
		return customers.purchases.length >= 3;
}

```

- 두번째 방법이 더 명확하고 재사용 하기에도 좋음
- 인라인 대신 이름을 붙여 콜백을 사용하면 단계가 중첩되는 것도 막을 수 있음

## 체이닝 팁 요약

- **데이터 만들기**
    - 함수형 도구는 배열 전체를 다룰 때 잘 동작함.
    - 배열 일부에 동작하는 반목문이 있다면 배열 일부를 새로운 배열로 나눌 수 있다.
- **배열 전체 다루기**
- **작은 단계로 나누기**
- 보너스
    - 조건문을 `filter()`로 바꾸기
    - 유용한 함수로 추출하기
    - 개선을 위해 실험하기

## 체이닝 디버깅을 위한 팁

- 구체적인 것을 유지하기
    - 각 단계에서 어떤일을 하고 있는지 알아보기 쉽게 이름을 지어야 함
- 출력해보기
    - 각 단계 사이에 print(or console) 찍어보면서 확인하기
- 타입을 따라가보기
    - 각 단계에서 만들어지는 값의 타입을 따라가면서 단계를 살펴보기

## 요점 정리

- 함수형 도구는 여러 단계의 체인으로 조합할 수 있음. → 복잡한 계산을 명확한 단계로 표현
- 함수형 도구 체인으로 배열을 다루는 복잡한 쿼리를 표현
- 체인의 다음단계를 위해 새로운 데이터를 만들거나 기존 데이터를 인자로 사용해야 하는 경우에는 최대한 명시적으로 표현하는 방법을 찾아야 함
