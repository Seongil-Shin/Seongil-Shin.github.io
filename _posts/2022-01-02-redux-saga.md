---
title: redux-saga concepts
author: 신성일
date: 2022-01-02 18:19:26 +0900
categories: [study, react]
tags: [redux]
---

## Declarative Effects

```javascript
import { takeEvery } from "redux-saga/effects";
import Api from "./path/to/api";

function* watchFetchProducts() {
  yield takeEvery("PRODUCTS_REQUESTED", fetchProducts);
}

function* fetchProducts() {
  const products = yield Api.fetch("/products");
  console.log(products);
}
```

```javascript
const iterator = fetchProducts()
assert.deepEqual(iterator.next().value, ??)
```

1. iterator 에는 제너레이터 객체가 반환됨.
2. next() 를 하면, Api.fetch 가 실행되면서 Promise 객체가 반환됨.

이때 위 featchProducts()를 테스트하기 위해선, API를 호출해야하는데, 실제로 호출할 순 없으므로 다른 방법을 통해 API를 호출하지 않고 테스트를 진행해야한다.

### effects 사용

```javascript
import { call } from "redux-saga/effects";

function* fetchProducts() {
  const products = yield call(Api.fetch, "/products");
  // ...
}
```

```javascript
import { call } from "redux-saga/effects";
import Api from "...";

const iterator = fetchProducts();

// expects a call instruction
assert.deepEqual(
  iterator.next().value,
  call(Api.fetch, "/products"),
  "fetchProducts should yield an Effect call(Api.fetch, './products')"
);
```

- 위에서와는 달리, iterator.next() 는 call 함수를 반환하게 됨.
- 실제 API를 호출하지 않으므로, 테스트하기가 훨씬 용이해진다.

### api 호출 effects

```javascript
yield call([obj, obj.method], arg1, arg2, ...);		// 주어진 object의 주어진 method를 호출
yield call(fn, arg1, arg2, ...);
```

```javascript
yield apply(obj, obj.method, [arg1, arg2, ...])  // call과 같고 arguments 형식만 반대
```

```javascript
yield cps(fn, arg1, arg2, ...)
```

- node 스타일 함수를 호출할때 사용 ((error, result) => ())

## Dispatching Actions

제너레이터 함수 안에서, dispatch를 하기 위해 아래와 같이 직접 dispatch를 호출할 수 있다.

```javascript
function* fetchProducts(dispatch) {
  const products = yield call(Api.fetch, "/products");
  dispatch({ type: "PRODUCTS_RECEIVED", products });
}
```

하지만 이는 위에서와 같이, 테스트를 하기 위해선 dispatch 함수를 mock 해야할 필요가 있다.

따라서 redux-saga에서는 아래와 같이 "put" effect를 통해 dispatch를 할 수 있다.

```javascript
import { call, put } from "redux-saga/effects";
// ...

function* fetchProducts() {
  const products = yield call(Api.fetch, "/products");
  // create and yield a dispatch Effect
  yield put({ type: "PRODUCTS_RECEIVED", products });
}
```

## Error handling

사가에서는 아래와 같은 방식으로 error handling 가능

```javascript
import Api from "./path/to/api";
import { call, put } from "redux-saga/effects";

// ...

function* fetchProducts() {
  try {
    const products = yield call(Api.fetch, "/products");
    yield put({ type: "PRODUCTS_RECEIVED", products });
  } catch (error) {
    yield put({ type: "PRODUCTS_REQUEST_FAILED", error });
  }
}
```

아래와 같은 형태로도 가능하다

```javascript
import Api from "./path/to/api";
import { call, put } from "redux-saga/effects";

function fetchProductsApi() {
  return Api.fetch("/products")
    .then((response) => ({ response }))
    .catch((error) => ({ error }));
}

function* fetchProducts() {
  const { response, error } = yield call(fetchProductsApi);
  if (response) yield put({ type: "PRODUCTS_RECEIVED", products: response });
  else yield put({ type: "PRODUCTS_REQUEST_FAILED", error });
}
```

## Saga helpers

- takeEvery : 모든 dispatch 액션을 수용함
- takeLatest : 가장 나중에 들어온 액션만 수용함
  - 이전에 들어와 수행중인 task는 중단되고, 나중에 들어온 task 가 수행되는 방식

```javascript
import { takeEverym, takeLatest } from "redux-saga/effects";

function* watchFetchData() {
  yield takeEvery("FETCH_REQUESTED", fetchData);
}
function* watchFetchData() {
  yield takeLatest("FETCH_REQUESTED", fetchData);
}
```

## channel

아래와 같이 take/fork 를 통해 여러 요청을 동시적으로 수행할 수 있다.

```javascript
import { take, fork, ... } from 'redux-saga/effects'

function* watchRequests() {
  while (true) {
    const {payload} = yield take('REQUEST')
    yield fork(handleRequest, payload)
  }
}

function* handleRequest(payload) { ... }
```

이때 REQUEST 요청을 순차적으로 첫번째부터 수행하고 싶을때는, queue를 만들어서 하나가 끝나면 다음 것을 꺼내어 수행하는 방식으로 하면 된다.

이때 redux-sage에서는 actionChannel(pattern)이라는 effect를 사용하여 구현할 수 있다. actionChannel은 saga가 API call 등으로 막혀있을때도 메세지를 받아 큐에 넣을 수 있다.

```javascript
import { take, actionChannel, call, ... } from 'redux-saga/effects'

function* watchRequests() {
  // 1- Create a channel for request actions
  const requestChan = yield actionChannel('REQUEST')
  while (true) {
    // 2- take from the channel
    const {payload} = yield take(requestChan)
    // 3- Note that we're using a blocking call
    yield call(handleRequest, payload)
  }
}

function* handleRequest(payload) { ... }
```

## composing sagas

watch에서 call을 yield하면 saga는 call이 끝날때까지 기다린다.

```javascript
function* fetchPosts() {
  yield put(actions.requestPosts());
  const products = yield call(fetchApi, "/products");
  yield put(actions.receivePosts(products));
}

function* watchFetch() {
  while (yield take("FETCH_POSTS")) {
    yield call(fetchPosts); // waits for the fetchPosts task to terminate
  }
}
```

다음과 같이 all 이펙트를 사용하여 여러 제너레이터를 동시에 수행하고, 모두 끝날때까지 기다릴 수도 있다.

```javascript
function* mainSaga(getState) {
  const results = yield all([call(task1), call(task2), ...])
  yield put(showResults(results))
}
```

## concurrency

takeEvery와 takeLatest가 어떻게 동시성을 제어하는가?

### takeEvery

```javascript
import { fork, take } from "redux-saga/effects";

const takeEvery = (pattern, saga, ...args) =>
  fork(function* () {
    while (true) {
      const action = yield take(pattern);
      yield fork(saga, ...args.concat(action));
    }
  });
```

해당하는 pattern이 들어오면 fork를 통해 분리하여 처리한다.

### takeLatest

```javascript
import { cancel, fork, take } from "redux-saga/effects";

const takeLatest = (pattern, saga, ...args) =>
  fork(function* () {
    let lastTask;
    while (true) {
      const action = yield take(pattern);
      if (lastTask) {
        yield cancel(lastTask); // cancel is no-op if the task has already terminated
      }
      lastTask = yield fork(saga, ...args.concat(action));
    }
  });
```

새로운 것이 들어오면 이전에 들어온 것을 취소하고, 새로운 것을 수행한다.

## Fork model

redux-saga에서는 다음 두 이펙트를 통해 fork를 다룬다

- fork : attached fork를 만듦
- spawn : datached fork를 만듦

### attached fork

Saga는 자기 안의 모든 명령어가 수행되고, 모든 attached forks가 종료되면 종료된다.

```javascript
import { fork, call, put, delay } from "redux-saga/effects";
import api from "./somewhere/api"; // app specific
import { receiveData } from "./somewhere/actions"; // app specific

function* fetchAll() {
  const task1 = yield fork(fetchResource, "users");
  const task2 = yield fork(fetchResource, "comments");
  yield delay(1000);
}

function* fetchResource(resource) {
  const { data } = yield call(api.fetch, resource);
  yield put(receiveData(data));
}

function* main() {
  yield call(fetchAll);
}
```

따라서 위 call(fetchAll)은 delay(1000)이 끝나고, fork 된 task1, task2 가 끝나야 종료된다.

이는 아래와 같이 parallel effect와 같은 동작을 한다.

```javascript
function* fetchAll() {
  yield all([
    call(fetchResource, "users"), // task1
    call(fetchResource, "comments"), // task2,
    delay(1000),
  ]);
}
```

이때 위 코드는 3개의 child effect 중 하나가 실패하면, Error가 넘어가면서 3개의 child가 전부 실패하고(완료되지 않은 작업들만), Error는 call로 fetchAll을 호출한 함수로 throw된다.

이는 fork를 이용한 모델에서도 똑같이 동작하며 아래와 같이 main에서 try/catch를 통해 에러를 받을 수 있다.

```javascript
//... imports

function* fetchAll() {
  const task1 = yield fork(fetchResource, "users");
  const task2 = yield fork(fetchResource, "comments");
  yield delay(1000);
}

function* fetchResource(resource) {
  const { data } = yield call(api.fetch, resource);
  yield put(receiveData(data));
}

function* main() {
  try {
    yield call(fetchAll);
  } catch (e) {
    // handle fetchAll errors
  }
}
```

### Cancellation

Saga를 cancel하면, 아래 것들이 cancel된다

- saga가 현재하고 있는 effect부터 cancel됨
- 모든 attached fork 가 cancle됨

### Detached forks

detached fork는 그 자체러 하나의 수행환경이다. 따라서, 부모는 detached fork의 종료를 기다리지 않고, 에러는 상위 부모로 올라가지 않는다.

또 cancle을 해도 detached fork는 영향을 받지 않는다.
