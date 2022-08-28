---
title: JS创建私有变量 weakmap
---

## JS创建私有变量属性

1.使用WeakMap

```js
const wm = new WeakMap();
const setPrivateProp = function(obj, prop, val){
	if(!wm.has(obj)) wm.set(obj,{})
	let v = wm.get(obj); 
	v[prop] = val
}
class User {
  constructor(name){
		this.setName(name);
  }
  getName(){
  	return wm.get(this).name;
  }
  setName(name){
  	setPrivateProp(this, "name", name);
  }
}
```

上面的案例还是可以访问到私有变量

```js
const user = new User("jay");
wm.get(user).name; //jay
```

2.闭包，将上面的案例用立即执行函数封装

```js
(function User(){
  const wm = new WeakMap();
  const setPrivateProp = function(obj, prop, val){
    if(!wm.has(obj)) wm.set(obj,{})
    let v = wm.get(obj); 
    v[prop] = val
  }
  class User {
    constructor(name){
      this.setName(name);
    }
    getName(){
      return wm.get(this).name;
    }
    setName(name){
      setPrivateProp(this, "name", name);
    }
  }
  return User
})()
```

