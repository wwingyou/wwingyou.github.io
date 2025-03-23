---
layout: post
title: 노출모듈 패턴
tags: DesignPattern
---

## 노출모듈 패턴

**JavaScript**와 같이 `private`, `public` 접근 제어자가 없는 언어에서 접근
제어자를 만들기 위해 사용하는 패턴이다. 즉시실행 함수를 이용해 데이터를 함수
스코프 안에 저장하여 외부에서 접근할 수 없게 만든 후, 노출할 데이터만 반환하는
방식이다.

```javascript
const store = (() => {
    const value = 1
    const public = {
        getValue : () => value
    }
    return public
})()

console.log(store.value)
console.log(store.getValue())
// undefined
// 1
```

CJS(CommonJS)가 이 방식을 사용한다.
