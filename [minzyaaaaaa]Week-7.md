# Chapter 14. 중첩된 데이터에 함수형 도구 사용하기

<aside>
💡 해시 맵에 저장된 값을 다루기 위한 고차 함수를 만든다.
중첩된 데이터를 고차 함수로 쉽게 다루는 방법을 배운다.
재귀를 이해하고 안전하게 재귀를 사용하는 방법을 살펴본다.
깊이 중첩된 엔티티에 추상화 벽을 적용해서 얻을 수 있는 장점을 이해한다.

</aside>

```jsx
// 중첩된 객체 예시
var shirt = {
	name: "shirt",
	price: 13,
	options: {
		color: "blue",
		size: 3
	}
}
```

```jsx
// Before
function incrementSize(item) {
	var options = item.options;
	var size = options.size;
	var newSize = size + 1;
	var newOptions = objectSet(options, 'size', newSize);
	var newItem = objectSet(item, 'options', newOptions);
	return newItem;
}
```

```jsx
// After-1 
function incrementSize(item) {
	var options = item.options;
	var newOptions = update(options, 'size', increment);
	var newItem = objectSet(item, 'options', newOptions);
	return newItem;
}
// After-2
function incrementSize(item) {
	return update(item, 'options', function(options) {
		return update(options, 'size', increment);
	});
}
```

`updateOptions()` 도출하기

```jsx
// Before
function incrementSize(item) {
	return update(item, 'options', function(options) {
		return update(options, 'size', increment);
	});
}
```

```jsx
// 암묵적 인자를 드러내기
// 1. 암묵적 option 인자
function incrementOption(item, option) {
	return update(item, 'options', function(options) {
		return update(options, option, increment);
	});
};

// 2. 암묵적 modify 인자
function updateOption(item, option, modify) {
	return update(item, 'options', function(options) {
		return update(options, option, modify);
	}
};
```

`update2()`도출하기

```jsx
// 여전히 필드명이 함수 이름에 있다 - option
function updateOption(item, option, modify) {
	return update(item, 'options', function(options) {
		return update(options, option, modify);
	}
};

function update2(object, key1, key2, modify) {
	return update(object, key1, function(value1) {
		return update(value1, key2, modify);
	});
}

// 같은 방식으로 update3()도 도출할 수 있다.
```

`nestedUpdate()` 도출하기

```jsx
function nestedUpdate(object, keys, modify) {
	// 0인 경우를 처리
	if(keys.length === 0) return modify(object);
	var key1 = keys[0];
	var restOfKeys = drop_first(keys);
	return update(object, key1, function(value1) {
		// 재귀 호출
		return nestedUpdate(value1, restOfKeys, modify);
	});
}
```

## 재귀함수

### 안전한 재귀 사용법

재귀는 `for`나 `while` 반복문처럼 무한 반복에 빠질 수 있다.

1. **종료 조건** 
2. **재귀 호출**
3. **종료 조건에 다가가기** - 재귀 호출에 같은 인자를 그대로 전달하면 무한 반복에 빠질 가능성이 높아진다.

## 깊이 중첩된 데이터에 추상화 벽 사용하기

```jsx
// 추상화 벽 함수 예제
// 주어진 ID로 블로그를 변경하는 함수
function updatePostById(category, id, modifyPost) {
	// 분류의 구조 ['posts', id] 같은 구체적인 부분은 추상화 벽 뒤로 숨김
	return nestedUpdate(category, ['posts', id], modifyPost);
}

function updateAuthor(post, modifyUser) {
	return update(post, 'author', modifyUser);
}

function captializeName(user) {
	return update(user, 'name', capitalize);
}

updatePostById(blogCategory, '12', function(post) {
	return updateAuthor(post, capitalizeUserName);
});
```

## Summary

- 보통 일반적인 반복문은 재귀보다 명확하다. 하지만 중첩된 데이터를 다룰 때는 재귀가 더 쉽고 명확하다.
- 재귀는 스스로 불리었던 곳이 어디인지 유지하기 위해 스택을 사용한다. 재귀 함수에서 스택은 중첩된 데이터 구조를 그대로 반영한다.
- 많은 키를 가지고 있는 깊이 중첩된 구조에 추상화 벽을 사용하면 알아야 할 것이 줄어든다. 추상화 벽으로 깊이 중첩된 데이터 구조를 쉽게 다를 수 있다.

# Chapter 15. 타임라인 격리하기

타임라인 : 액션을 순서대로 나열한 것

타임라인 다이어그램 : 시간에 따른 액션 순서를 시각적으로 표시한 것

### 타임라인 다이어그램 기본 규칙

1. 두 액션이 순서대로 나타나면 같은 타임라인에 넣는다.
    
    타임라인에는 액션만 그린다. 계산은 실행 시점에 영향을 받지 않기 때문에 그리지 않는다.
    
2. 두 가지 액션이 동시에 실행되거나 순서를 예상할 수 없다면 분리된 타임라인에 넣는다.
    
    액션이 서로 다른 스레드나 프로세스, 기계, 비동기 콜백에서 실행되면 서로 다른 타임라인에서 표시한다. 이 경우는 액션 두 개가 서로 다른 비동기 콜백에서 실행된다. 두 액션의 실행 시점이 무작위이기 때문에 어떤 액션이 먼저 실행될지 알 수 없다.
    
3. 비동기 호출은 새로운 타임라인으로 그린다.

**요약**

1. 액션은 순서대로 실행되거나 동시에 실행된다.
2. 순서대로 실행되는 액션은 같은 타임라인에서 하나가 끝나면 다른 하나가 실행된다.
3. 동시에 실행되는 액션은 여러 타임라인에서 나란히 실행된다.

### 언어에 따라 다른 스레드 모델

1. 단일 스레드, 동기

모든 것이 **순서대로 실행**되고 입출력을 사용하면 끝날 때까지 기다려야 한다. 시스템이 단순하다는 장점이기도 하다. 스레드가 하나면 타임라인도 하나이지만, 네트워크를 통한 API 호출 같은 것은 다른 타임라인이 필요하다. 하지만 메모리를 공유하지 않기 때문에 공유 자원을 많이 없앨 수 있다.

2. 단일 스레드, 비동기

자바스크립트는 스레드가 하나이다. 입출력 작업을 하려면 비동기 모델을 사용해야 한다. 입출력의 결과는 콜백으로 받을 수 있지만, 언제 끝날지 알 수 없기 때문에 다른 타임라인에 표현해야 한다.

3. 멀티스레드

멀티스레드는 실행 순서를 보장하지 않기 때문에 프로그래밍 하기 매우 어렵다. 새로운 스레드가 생기면 새로운 타임라인을 그려야한다.

4. 메시지 패싱 (message-passing) 프로세스

엘릭서나 얼랭 같은 언어는 서로 다른 프로세스를 동시에 실행할 수 있는 스레드 모델을 지원한다. 프로세스는 서로 메모리를 공유하지 않고 메시지로 통신한다. 서로 다른 타임라인에 있는 액션은 순서가 섞이지만, 메모리를 공유하지 않기 때문에 가능한 실행 순서가 많아도 문제가 되지 않는다.

### 자바스크립트의 단일 스레드

1. **자바스크립트의 단일 스레드**
- 자바스크립트는 스레드가 하나이다.
- 전역변수를 바꾸는 동기 액션은 타임라인이 서로 섞이지 않는다.
- 비동기 호출은 미래에 알 수 없는 시점에 런타임에 의해 실행된다.
- 두 동기 액션은 동시에 실행되지 않는다.

2. **자바스크립트의 단일 큐**

브라우저에서 동작하는 자바스크립트 엔진은 **작업 큐** 라는 큐를 가지고 있다. 작업 큐는 **이벤트 루프**에 의해 처리된다. 이벤트 루프는 큐에서 작업하나를 꺼내 실행하고 완료되면 다음 작업을 꺼내 실행하는 것을 무한히 반복한다. 이벤트 루프는 하나의 스레드에서 처리하기 때문에 두 개의 작업이 동시에 실행될 수 없다.

작업이란?

작업 큐에 있는 작업은 이벤트 데이터와 이벤트를 처리할 콜백으로 구성되어 있다. 이벤트 루프는 이벤트 데이터를 인자로 콜백을 부른다. 콜백은 이벤트 루프가 실행할 함수이다. 

이벤트가 발생하면 큐에 작업이 추가된다. 만약 콜백 함수가 있는 버튼에 이벤트가 발생하면 콜백 함수와 이벤트 데이터가 큐에 추가된다. 이벤트는 예상할 수 없기 때문에 예측 불가능한 시점에 작업 큐에 들어간다.

### AJAX

Asynchronous JavaScript And XML / AJAX는 브라우저에 기반을 둔 웹 요청을 말한다. 

자바스크립트에서 AJAX 요청을 만들면 네트워크 엔진이 AJAX 요청을 처리하기 위해 요청 큐에 넣는다. 요청 큐에 작업이 추가되어도 코드는 계속 실행된다. 네트워크 환경은 예측할 수 없기 때문에 응답은 요청 순서대로 오지 않는다.

## Summary

- 현대 소프트웨어는 여러 타임라인에서 실행된다.
- 서로 다른 타임라인에 있는 액션은 여러 개의 실행 순서가 생길 수 있다. 실행 가능한 순서가 많다는 것은 코드가 항상 예측대로 실행되지 않을 수 있다는 의미이다.
- 타임라인 다이어그램은 코드가 순서대로 실행되는지, 동시에 실행되는지 알려준다. 타임라인 다이어그램을 통해 서로 영향을 주는 부분을 파악할 수 있다.
- 언어에서 지원하는 스레드 모델을 이해하는 것은 중요하다.
- 자원을 공유하는 부분은 버그가 발생하기 쉽다. 공유 자원을 확인하고 없앤다면, 더 좋은 코드를 작성할 수 있다. 자원을 공유하지 않는 타임라인은 독립적으로 이해하고 실행할 수 있다.
