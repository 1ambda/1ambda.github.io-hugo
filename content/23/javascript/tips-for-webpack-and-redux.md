+++
date = "2016-06-25T12:24:38+09:00"
prev = "../"
title = "Tips for Webpack and Redux"
toc = true
weight = 12
aliases = [
    "/tips-redux-and-webpack"
]
+++

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/redux/redux_logo.png?width=30%&height=30%)

지난 한달 동안 자그마한 웹앱 프로젝트를 Redux 를 이용해서 진행했습니다. 그 과정에서 배운 몇 가지를 적었습니다.

[Redux: 1. combineReducers 를 이용해 Reducer 를 잘게 분해하기](#1combinereducersreducer)  
[Redux: 2. Reducer 에서는 관련있는 Action 만 처리하기](#2reduceractionactiontype)  
[Redux: 3. redux internal 이해하기](#3reduxinternal)  
[Redux: 4. redux-saga 사용하기](#4reduxsaga)  
[Redux: 5. API 호출 실패에 대한 액션을 여러개 만들지 않기](#5api)  
[Webpack: 6. 테스팅 프레임워크로 jest 대신 mocha 사용하기](#6jestmocha)  
[Webpack: 7. postcss 를 사용할 경우, 테스팅 환경에서 스타일파일 무시하기](#7postcss)  
[Webpack: 8. DefinePlugin 을 이용해 클라이언트 파일에 환경변수 주입하기](#8defineplugin)  
[Etc: 9. json-server 사용하기](#9jsonserver)  

## Redux

### 1. combineReducers 를 이용해 Reducer 를 잘게 분해하기

Root Reducer 가 `JobReducer` 를 포함하고 있고, `JobReducer` 는 Job 과 관련된 모든 상태를 다룬다고 할 때 다음처럼 `combineReducers` 를 이용해서 `JobReducer` 를 분해하면, 개별 컴포넌트의 상태(*State*) 는 각각의 서브 리듀서 (이하 핸들러) 에서 다루면 됩니다.

```javascript
// https://github.com/1ambda/slott/blob/master/src/reducers/JobReducer/index.js

import { combineReducers, } from 'redux'

import * as JobItemState from './JobItemState'
import * as PaginatorState from './PaginatorState'
import * as FilterState from './FilterState'
import * as SorterState from './SorterState'
...

export const JOB_STATE_PROPERTY = {
  JOB_ITEMS: 'items',
  PAGINATOR: 'paginator',
  FILTER: 'filterKeyword',
  SORTER: 'sortingStrategy',
  ...
}

export default combineReducers({
  [JOB_STATE_PROPERTY.CONTAINER_SELECTOR]: ContainerSelectorState.handler,
  [JOB_STATE_PROPERTY.JOB_ITEMS]: JobItemState.handler,
  [JOB_STATE_PROPERTY.PAGINATOR]: PaginatorState.handler,
  [JOB_STATE_PROPERTY.FILTER]: FilterState.handler,
  [JOB_STATE_PROPERTY.SORTER]: SorterState.handler,
  ...
})
```

```javascript
// https://github.com/1ambda/slott/blob/master/src/reducers/JobReducer/FilterState.js

import { createAction, handleActions, } from 'redux-actions'

const INITIAL_STATE = '' /** initial state of FilterState */

export const ActionType = {
  FILTER: 'JOB_FILTER',
  INITIALIZE_FILTER: 'JOB_INITIALIZE_FILTER',
}

export const Action = {
  filterJob: createAction(ActionType.FILTER),
  initializeFilter: createAction(ActionType.INITIALIZE_FILTER),
}

export const handler = handleActions({
  [ActionType.FILTER]: (state, { payload, }) =>
    payload.filterKeyword, /** since string is immutable. we don't need to copy old state */

  [ActionType.INITIALIZE_FILTER]: (state, { payload, }) =>
    INITIAL_STATE,
}, INITIAL_STATE)
```

### 2. Reducer, Action, ActionType 을 한 파일로 모으기

[Redux Github]() 에 나와있는 예제에서는 `ActionType` 과 `Action` 을 하나의 파일에 모아놓는데, 프로젝트가 커질수록 부담스럽습니다.

`Action`, `ActionType`, `Handler` 를 한 파일에 모아놓으면 이 핸들러가 어떤 일들을 하는지, 페이로드는 무엇인지 한 눈에 파악할 수 있습니다. 

```javascript
// https://github.com/1ambda/slott/blob/master/src/reducers/JobReducer/PaginatorState.js

import { createAction, handleActions, } from 'redux-actions'

import { PAGINATOR_ITEM_COUNT_PER_PAGE, } from '../../constants/config'

import * as FilterState from './FilterState'
import * as SorterState from './SorterState'

const INITIAL_PAGINATOR_STATE = {
  currentPageOffset: 0,
  currentItemOffset: 0,
  itemCountPerPage: PAGINATOR_ITEM_COUNT_PER_PAGE,
}

export const ActionType = {
  CHANGE_PAGE_OFFSET: 'JOB_CHANGE_PAGE_OFFSET',
}

export const Action = {
  changePageOffset: createAction(ActionType.CHANGE_PAGE_OFFSET),
}

export const handler = handleActions({
  [ActionType.CHANGE_PAGE_OFFSET]: (state, { payload, }) => {
    const { newPageOffset, } = payload
    const currentItemOffset = newPageOffset * state.itemCountPerPage
    return Object.assign({}, state, {currentPageOffset: newPageOffset, currentItemOffset,})
  },

  /** reset paginator if filter or sorter action is occurred */
  [SorterState.ActionType.SORT]: (state) => INITIAL_PAGINATOR_STATE,
  [FilterState.ActionType.FILTER]: (state) => INITIAL_PAGINATOR_STATE,
}, INITIAL_PAGINATOR_STATE)
```

`Paginator` 가 어떤 액션을 처리하고, 페이로드는 무엇인지 하나의 파일에서 확인할 수 있습니다.

### 3. redux internal 이해하기

[redux](https://github.com/reactjs/redux) 의 놀라운 점중 하나는 소스코드가 길지 않다는 점입니다. 따라서 내부 구조를 이해하기도 어렵지 않은데요,

[Redux Middleware: Behind the Scenes](http://briantroncone.com/?p=529) 를 참고하면, *enhancer* 가 어떻게 조합되고, *store* 가 어떻게 생성되는지 쉽게 알 수 있습니다.

### 4. redux-saga 사용하기

[redux-saga](https://github.com/yelouafi/redux-saga) 를 이용하면 `Promise` 가 들어가는 비동기 로직을 [ES7 async](https://tc39.github.io/ecmascript-asyncawait/) 를 이용하는것처럼 작성할 수 있습니다. 추가적으로 사이드이펙트 (e.g API call) 의 선언과 실행 시점을 분리해 테스트를 쉽게 할 수 있도록 도와줍니다.

예를 들어, 초기화 시점에 서버로부터 전체 Job 을 가져오는 로직을 [redux-saga](https://github.com/yelouafi/redux-saga) 를 이용해 다음처럼 작성할 수 있습니다.

```javascript
// https://github.com/1ambda/slott/blob/master/src/middlewares/sagas.js#L12

import { fork, call, put, } from 'redux-saga/effects'

import * as SnackbarState from '../reducers/JobReducer/ClosableSnackbarState'
import * as Handler from './handler'

export function* initialize() {
  try {
    yield call(Handler.callFetchContainerJobs)
  } catch (error) {
    yield put(
      SnackbarState.Action.openErrorSnackbar(
        { message: 'Failed to fetch jobs', error, }
      )
    )
  }
}
```

위 코드는 서버로부터 모든 Job 을 가져오고, 그 과정에서 예외가 발생하면 Snackbar 에 예외메세지를 출력하는 Action 을 Reducer 로 보내는 코드입니다. (여기서 `Handler.callFetchContainerJobs` 가 `Promise` 를 돌려준다고 보고)

이 때 다음처럼 테스트를 작성할 수 있습니다.

```javascript
// https://github.com/1ambda/slott/blob/master/src/middlewares/__tests__/sagas.spec.js#L87

  describe('initialize', () => {
    it('should callFetchContainerJobs', () => {
      const gen = Sagas.initialize()
      expect(gen.next().value).to.deep.equal(
        call(Handler.callFetchContainerJobs)
      )

      expect(gen.next().done).to.deep.equal(true)
    })

    it(`should callFetchJobs
        - if exception is occurred,
          put(openErrorSnackbar with { message, error }`, () => {
      const gen = Sagas.initialize()

      expect(gen.next().value).to.deep.equal(
        call(Handler.callFetchContainerJobs)
      )

      const error = new Error('error')
      expect(gen.throw(error).value).to.deep.equal(
        put(ClosableSnackBarState.Action.openErrorSnackbar({ message: 'Failed to fetch jobs', error, }))
      )
    })
  })
```

위 테스트 코드에서 알 수 있듯이, `redux-saga/effects` 의 `call` 을 호출하는 시점에서 AJAX 이 실행되지 않습니다. 실제로는 `call` 은 AJAX 실행할것임을 **선언** 만 합니다. AJAX 은 `call` 로 부터 생성된 redux 액션이 `redux-saga` 미들웨어에서 처리되는 순간에 **실행** 됩니다. `call` 의 리턴값은, 어떤 redux 액션이 실행될 것인지 알려주는 자바스크립트 객체입니다. 위에서는 이 리턴값을 이용해 테스트를 작성한 것입니다.

```
// https://github.com/yelouafi/redux-saga/blob/master/docs/basics/DeclarativeEffects.md


{
  CALL: {
    fn: Handler.callFetchContainerJobs,
    args: []  
  }
}
```

### 5. API 호출 실패에 대한 액션을 여러개 만들지 않기

redux 나 [redux-saga 예제](https://github.com/yelouafi/redux-saga/blob/ce1d701467d2ec4e8c5c40288d9a41254c6f3583/examples/real-world/actions/index.js) 를 보면, API 실패에 대한 액션을 여러 종류로 만드는 것을 알 수 있습니다.

그러나 일반적으로 예외는 단일화된 방식으로 (e.g 에러 다이어로그, 팝업, 페이지 등) 처리되기 때문에 에러를 다룰 UI 컴포넌트에 대한 1개의 액션만 만드는 것이 더 바람직 합니다. 예를 들어 Snackbar 에서 예외 메세지를 보여준다고 할 때 다음처럼 액션 핸들러를 작성할 수 있습니다.

```javascript
// https://github.com/1ambda/slott/blob/e2fc9c1260a5c8202ad747c31f5907ff29ab9a94/src/reducers/JobReducer/ClosableSnackbarState.js#L27

export const handler = handleActions({
  /** snackbar related */
  [ActionType.CLOSE_SNACKBAR]: (state) =>
    Object.assign({}, state, { snackbarMode: CLOSABLE_SNACKBAR_MODE.CLOSE, }),

  [ActionType.OPEN_ERROR_SNACKBAR]: (state, { payload, }) =>
    Object.assign({}, state, {
      snackbarMode: CLOSABLE_SNACKBAR_MODE.OPEN,
      message: `[ERROR] ${payload.message} (${payload.error.message})`,
    }),

  [ActionType.OPEN_INFO_SNACKBAR]: (state, { payload, }) =>
    Object.assign({}, state, {
      snackbarMode: CLOSABLE_SNACKBAR_MODE.OPEN,
      message: `[INFO] ${payload.message}`,
    }),

}, INITIAL_SNACKBAR_STATE)
```

만약 여러 종류의 API 실패에 대한 액션을 처리하도록 작성했다면, 이런 코드가 되었을 거고 API_FAILED 액션 타입이 삭제되고 추가될 때 마다 수정해야 하므로 변경에 취약했을 것입니다.

```javascript

const FAILED_API_ACTION_TYPES = [
  ActionType.LOAD_ALL_JOBS_FAILED,
  ActionType.CREATE_JOB_FAILED,
  ActionType.REMOVE_JOB_FAILED,
  ...
]

const FailureHandlers = FAILED_API_ACTION_TYPES.map(actionType => {
  return { [actionType]: (state, { payload, }) =>
    Object.assign({}, state, {
      snackbarMode: CLOSABLE_SNACKBAR_MODE.OPEN,
      message: `[ERROR] ${payload.message} (${payload.error.message})`,
    })
  }
})

export const handler = handleActions({
  /** snackbar related */
  [ActionType.CLOSE_SNACKBAR]: (state) =>
    Object.assign({}, state, { snackbarMode: CLOSABLE_SNACKBAR_MODE.CLOSE, }),

  ...FailureHandlers,

}, INITIAL_SNACKBAR_STATE)
```

## Webpack

### 6. 테스팅 프레임워크로 jest 대신 mocha 사용하기

[jest](https://facebook.github.io/jest/) 는 Facebook 에서 만든 테스팅 프레임워크입니다. 모든 `import` 는 기본적으로 mocking 됩니다. 따라서 테스트할 `.js` 파일에서 사용되는 모든 라이브러리도 mocking 됩니다. 이런식으로 테스트 대상만 unmocking 해서 사용할 수 있습니다.

```javascript
// https://github.com/facebook/jest

jest.unmock('../sum'); // unmock to use the actual implementation of sum

describe('sum', () => {
  it('adds 1 + 2 to equal 3', () => {
    const sum = require('../sum');
    expect(sum(1, 2)).toBe(3);
  });
});
```

[jest](https://github.com/facebook/jest) 사용시 주의 할 사항이 두 가지 있습니다.

jest 0.9.0 기준으로 아직 모든 라이브러리가 mocking 되진 않습니다. (e.g redux-saga)
babel 을 사용할 경우 babel-jest 로 테스트 실행이 가능하지만 여기에 postcss 까지 같이 쓸 경우, `import (‘*.css)` 구문 때문에 테스팅이 불가능합니다. 커스텀 jest 로더를 등록하면, babel-runtime 로딩이 제대로 안되며 [webpack-babel-jest](https://github.com/atecarlos/webpack-babel-jest) 란것도 있으나 제대로 동작하지 않습니다. (관련이슈 [jest issue: 334 - How to test with Jest when I'm using webpack](https://github.com/facebook/jest/issues/334))

### 7. postcss 를 사용할 경우, 테스팅 환경에서 스타일파일 무시하기

[postcss](https://github.com/postcss/postcss) 를 이용하면 [autoprefixer](https://github.com/postcss/autoprefixer) 등의 각종 플러그인을 사용 가능합니다. 특히 [postcss-loader](https://github.com/postcss/postcss-loader) 를 이용하면
 지엽적인 css 클래스 생성과 적용이 가능하므로 모듈, 컴포넌트 단위로 관리되는 React 와 같이 쓰기 좋습니다.

그런데, 테스팅 환경에서는 [webpack](https://webpack.github.io/) 이 돌지 않으므로 css 파일 임포트가 불가능 하고, 테스트 실행이 안됩니다. 이 경우 [ignore-styles](https://www.npmjs.com/package/ignore-styles) 를 이용하거나 mocha 설정을 이용해 css 파일 임포트 문장을 무시할 수 있습니다.

```javascript
// https://github.com/1ambda/slott/blob/f1e94a9e693c30f466d92a4d3c988b75d5db4118/package.json#L28

"test": "cross-env NODE_ENV=test mocha --reporter progress --compilers js:babel-core/register --recursive \"./src/**/*.spec.js\" --require ignore-styles"
```

아니면 [react-slingshot](https://github.com/coryhouse/react-slingshot) 처럼 셋업 파일을 분리해서 mocha 설정으로 이용할 수 있습니다.

```javascript
// https://github.com/coryhouse/react-slingshot/blob/16ec28c9029bf7e2b65b26c22a1c2daadab427a2/tools/testSetup.js

process.env.NODE_ENV = 'production';

// Disable webpack-specific features for tests since
// Mocha doesn't know what to do with them.
require.extensions['.css'] = function () {
  return null;
};
require.extensions['.png'] = function () {
  return null;
};
require.extensions['.jpg'] = function () {
  return null;
};

// Register babel so that it will transpile ES6 to ES5
// before our tests run.
require('babel-register')();
```

이후 `package.son` 에서 `"test": "mocha tools/testSetup.js src/**/*.spec.js --reporter ` 처럼 사용할 수 있습니다.

### 8. DefinePlugin 을 이용해 클라이언트 파일에 환경변수 주입하기

[Webpack: DefinePlugin](https://webpack.github.io/docs/list-of-plugins.html#defineplugin) 을 이용하면 Webpack 실행 시점에 존재하는 변수를 클라이언트에 주입할 수 있습니다. (e.g 환경변수, 별도 파일로 존재하는 설정값 등) 예를 들어

```javascript
// https://github.com/1ambda/slott/blob/f1e94a9e693c30f466d92a4d3c988b75d5db4118/tools/config.js

import { ENV_DEV, ENV_PROD, ENV_TEST, } from './env'
import * as DEV_CONFIG from '../config/development.config'
import * as PROD_CONFIG from '../config/production.config'

const env = process.env.NODE_ENV

export const CONFIG = (env === ENV_DEV) ? DEV_CONFIG : PROD_CONFIG

export const GLOBAL_VARIABLES = { /** used by Webpack.DefinePlugin */
  'process.env.ENV_DEV': JSON.stringify(ENV_DEV),
  'process.env.ENV_PROD': JSON.stringify(ENV_PROD),
  'process.env.NODE_ENV': JSON.stringify(env),

  /** variables defined in `CONFIG` file ares already stringified */
  'process.env.CONTAINERS': CONFIG.CONTAINERS,
  'process.env.TITLE': CONFIG.TITLE,
  'process.env.PAGINATOR_ITEM_COUNT': CONFIG.PAGINATOR_ITEM_COUNT,
}
```

좌측이 클라이언트에서 사용할 변수, 우측이 주입할 변수입니다. 이렇게 만든 후 Webpack 설정에서 다음처럼 사용할 수 있습니다.

```javascript
// https://github.com/1ambda/slott/blob/f1e94a9e693c30f466d92a4d3c988b75d5db4118/webpack.config.js

const getPlugins = function (env) {
  const plugins = [
    new webpack.optimize.OccurenceOrderPlugin(),
    new webpack.DefinePlugin(GLOBAL_VARIABLES),
    ...
  ]

  /* eslint-disable no-console */
  console.log('Injecting Global Variable'.green)
  console.log(GLOBAL_VARIABLES)
  /* eslint-enable no-console */

...
```

이 때 몇 가지 주의할 사항이 있습니다. 

> If the value is a string it will be used as a code fragment.
> If the value isn’t a string, it will be stringified (including functions).
> If the value is an object all keys are defined the same way.
> If you prefix typeof to the key, it’s only defined for typeof calls.

따라서 우측 값이 문자열일 경우, 코드값으로 사용되므로 `undefined` 로 주입되거나, Webpack 실행시 예외가 발생할 경우는 확인해 보아야 합니다. 실제로 문자열을 주입하고 싶다면 한번 더 문자열로 감싸야 하구요. 이 부분은 문서에도 나와 있습니다.

```javascript
// https://webpack.github.io/docs/list-of-plugins.html#defineplugin

new webpack.DefinePlugin({
    VERSION: JSON.stringify("5fa3b9"),
    BROWSER_SUPPORTS_HTML5: true,
    TWO: "1+1",
    "typeof window": JSON.stringify("object")
})
```

## Etc

### 9. json-server 사용하기

웹 클라이언트 개발 과정에서, API 연동을 하다보면 두 가지 문제점에 마주칩니다.

- **아직 백엔드가 개발되지 않았는데 연동이 필요할 경우**: 테스팅은 mock 등을 어찌어찌 해서 짤 수 있으나, UI 시뮬레이션은 최소한 로컬호스트 개발용 서버라도 갖추어야 하므로 어려움
- **RESTful API 구현**: HTTP Status, Methods, URI 등에 대한 학습과 고민이 필요

로컬에서 미리 정의된 리소스를 읽어 표준화된 REST API 서버를 제공하는 [json-server](https://github.com/typicode/json-server) 를 이용하면 이 두 가지 문제를 해결할 수 있습니다. 

예를 들어 Job 을 `/api/jobs` 에서 돌려준다고 하면 리소스 파일을 다음처럼 작성할 수 있습니다.

```json
{
  "jobs": [
    {
      "id": "akka-cluster-A-1",
      "tags": [
        "cluster"
      ],
      "active": true,
      "enabled": true,
      "kafka": {
        "topic": "akka-A",
        "consumer-group": "cluster-consumers"
      },
      "hdfs": "/data/akka/cluster-A"
    }
  , ...
  ]
}
```

추가적으로 라우팅 세팅을 위해 `routes.json` 파일을 다음처럼 작성하면 됩니다.

```json
{
  "/api/": "/",
```

[json-server](https://github.com/typicode/json-server) 를 사용할 때 두 가지 주의해야 할 점이 있습니다.

**1. `id` 값은 *immutable* 이고, 모든 리소스는 `id` 값을 가지고 있어야 합니다. (키 값은 `--id` 옵션으로 변경 가능함)**

따라서 각 Job 의 실행 상태와 설정값을 별개의 리소스가 아니라 (별개의 리소스라면 `jobId` 를 주어 *join* 을 해야함) `/api/jobs/:id/state`, `/api/jobs/:id/config` 처럼 nested 된 형태로 돌려주고 싶을 때는 `routes.json` 의 라우팅 트릭을 이용할 수 있습니다.

```json
// https://github.com/1ambda/slott/blob/f1e94a9e693c30f466d92a4d3c988b75d5db4118/resource/routes.json

{
  "/api/": "/",
  "/:resource/:id/state": "/:resource/:id",
  "/:resource/:id/config": "/:resource/:id"
}
```

이 때, 이 리소스는 별개의 리소스가 아니라 URI 만 매핑된 것이므로 `config`, `state` 등에 대한 변경은 HTTP *PATCH* 메소드로 변경해야 합니다.


**2. 모든 리소스 변경은 즉시 파일에 변경됩니다.**

따라서 매 실행마다 동일한 리소스로 시작하려면, 리소스 파일을 복사 후 실행하는 간단한 스크립트를 작성하면 됩니다.

```javascript
// https://github.com/1ambda/slott/blob/f1e94a9e693c30f466d92a4d3c988b75d5db4118/tools/remote.js

import fs from 'fs-extra'

/** initialize resource/remote/db.json */

const resourceDir = 'resource'

// 3개의 서버를 별개로 띄우므로 3벌 복사
const remotes = ['remote1', 'remote2', 'remote3',]

remotes.map(remote => {
  fs.copySync(`${resourceDir}/${remote}/db.origin.json`, `${resourceDir}/${remote}/db.json`)
})
```

이후 `package.json` 에 다음의 스크립트를 작성하고, 사용하면 됩니다.

```json
// https://github.com/1ambda/slott/blob/f1e94a9e693c30f466d92a4d3c988b75d5db4118/package.json#L9

...

 "start:mock-server1": "json-server resource/remote1/db.json --routes resource/routes.json --port 3002",
    "start:mock-server2": "json-server resource/remote2/db.json --routes resource/routes.json --port 3003",
    "start:mock-server3": "json-server resource/remote3/db.json --routes resource/routes.json --port 3004",
    "start:mock-server": "npm-run-all --parallel start:mock-server1 start:mock-server2 start:mock-server3",

...
```

## References

- [Redux Logo](https://github.com/reactjs/redux/issues/151)
- [Redux Middleware: Behind the Scenes](http://briantroncone.com/?p=529)
- [ES7 async](https://tc39.github.io/ecmascript-asyncawait/)
- [redux-saga: Declarative Effects](https://github.com/yelouafi/redux-saga/blob/master/docs/basics/DeclarativeEffects.md)
- [Webpack: DefinePlugin](https://webpack.github.io/docs/list-of-plugins.html#defineplugin)
- [react-slingshot](https://github.com/coryhouse/react-slingshot)
