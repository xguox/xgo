---
title: "Redux 的中 Store 的默认 state"
date: 2017-07-24T16:01:23+08:00
draft: false
tags: ["JavaScript", "React"]
categories: ["JavaScript", "React"]
---

写了一段时间 React, Redux, 发现之前对 Redux 的 Store 理解偏差了.

每次发生 dispatch 某一个 action, 除去 action.type 的对应的 reducer, 其余都返回默认值通常是 switch 的 default 语句, Redux 在背后做了一些事情, 并不是每次传的都是定义时候传的默认值, 而是把对应的 reducer 的改动后的 state 作为参数传给了这个 reducer

比如, 下面这个 reducer, 初始状态 state 等于 `{isLoading: false, isLoggedIn: false, token: localStorage.token}`, 假设 **dispatch 了 AUTH_SUCCESS** 以后, `{isLoading: false, isLoggedIn: true, token: localStorage.token,}` 就会存在了 Redux 的 Store 里边, 下一次如果 dispatch 的 action 只要对应的 action.type 不在这里边的任意一个, 自然会返回 **default** 值, 但是, 此时传给 **auth** 的默认 state 参数不在是定义方法时候的这个参数, 而是之前保存下来的 `{isLoading: false, isLoggedIn: true, token: localStorage.token,}`


```javascript
import {AUTH_FAILURE, AUTH_REQUEST, AUTH_SUCCESS} from '../actions/auth_actions';

export default function auth(state = {
  isLoading: false,
  isLoggedIn: false,
  token: localStorage.token,
}, action) {
  switch (action.type) {
    case AUTH_REQUEST:
      return Object.assign({}, state, {
        isLoading: true,
        isLoggedIn: false,
      });
    case AUTH_SUCCESS:
      return Object.assign({}, state,
          {
            isLoading: false,
            isLoggedIn: true,
            token: localStorage.token,
          });
    case AUTH_FAILURE:
      return Object.assign({}, state,
          {
            isLoading: false,
            isLoggedIn: false,
            errMsg: action.msg,
          });
    default:
      return state;
  }
};
```

当**浏览器刷新**以后, Store 的值就会被清掉, 再次 dispatch 则回归到方法定义时候的 state 参数了.

= . =

尴尬, 之前都理解成什么了.