## Leetcode
![image](https://note.youdao.com/yws/res/8760/9DBF832BFAAD4808BF2410EE7F54902A)

## Review
这周主要在学习typescript, 类型检查是多人合作的利器, 可和现在框架react的结合还是碰到了一些问题
https://www.typescriptlang.org/docs/handbook/advanced-types.html

## Tip
redux 中间件原理实现思路
![image](https://note.youdao.com/src/AB0880FC8686424F9613EC8262F7A766)
上图中fn4为目标要执行的函数, fn1 fn2 fn3为外层包裹代码, 代码执行顺序为
fn1 => fn2 => fn3 => fn4 => fn3 => fn2 => fn1

### 实现一个层级调用
fn1 => fn2 => fn3
```javascript
function fn1(fn) {
  console.log('第一层')
  console.log('fn是', fn)
  if (typeof fn === 'function') {
    fn()
  }
}
function fn2(fn) {
  console.log('第二层')
  console.log('fn是', fn)
  if (typeof fn === 'function') {
    fn()
  }
}
function fn3(fn) {
  console.log('第三层')
  console.log('fn是', fn)

  if (typeof fn === 'function') {
    fn()
  }
}
fn3(fn2(fn1()))
```

### 实现一个写死的完整的层级
要做到这个， 必须在一个函数里面加入另外一个函数的引用, 比如在fn1加入fn2的引用, 可以通过高级函数在包一层来实现， 下面函数的next就是表示这个函数引用， 需要用到函数的**柯里化**
```javascript
const fn1 = next => args => {
  console.log('fn1进')
  next(args)
  console.log('fn1出')
}
const fn2 = next => args => {
  console.log('fn2进')
  next(args)
  console.log('fn2出')
}
const fn3 = next => args => {
  console.log('fn3进')
  next(args)
  console.log('fn3出')
}

const fn4 = args => {
  console.log('到达fn4', args)
} 

const innerFn3 = fn3(fn4)
const innerFn2 = fn2(innerFn3)
const innerFn1 = fn1(innerFn2)

innerFn1()
```
注意参数的层层传递, 从最外层接收的参数最终要传到最里面
### 将函数组合过程封装
```javascript
const innerFn3 = fn3(fn4)
const innerFn2 = fn2(innerFn3)
const innerFn1 = fn1(innerFn2)
```
上述的函数是: 
传入一个结果(开头是fn4)，和数组的一个元素, 经过一个规则运算,得出一个结果,<br/> 这个结果和数组的第二个元素, 经过相同的规则, 得到一个结果
...循环往复

上面的规则符合reduce的用法: fn4就是reduce的第二个参数initalValue
```javascript
function compose(...funcs) {
  funcs.reverse()
  return funcs.reduce((rs, currentFn) => currentFn(rs))
}

const fn = compose(fn1, fn2, fn3, fn4)
fn('this is some payload send to the deepest level')
// 到达fn4 this is some payload send to the deepest level
```
注意需要reverse, 因为要做到这个结果, 实际顺序应该是: fn3传入fn2形成一个函数, fn2传入fn1形成一个新函数 ..., 参见上面写死代码的顺序

### 结合redux
模拟redux的api, 中间件-就是上面层层包裹的函数, 要可以在函数体里面调用redux的api, 需要再次用到函数柯里化
```javascript
const store = {
  getState() {
    return {
      name: 'Huaite',
      age: '25'
    }
  },
  dispatch(action) {
    console.log('set action to redux, action is', action)
  }
}
```

中间件在包一层
```javascript
const middleware1 = ({getState, dispatch}) => next => action => {
  console.log('fn1进, do something...')
  next(action)
  console.log('fn1出, do something...')
}
const middleware2 = ({getState, dispatch}) => next => action => {
  console.log('fn2进, do something...')
  next(action)
  console.log('fn2出, do something...')
}
const middleware3 = ({getState, dispatch}) => next => action => {
  console.log('fn3进, do something...')
  console.log('fn3', 'getState', getState())
  console.log('fn3', 'dispatch', dispatch)
  next(action)
  console.log('fn3出, do something...')
}
```

实现
```javascript
const middlewares = [middleware1, middleware2, middleware3]
const chains = middlewares.map(middleware => middleware(store))
chains.push(store.dispatch) // 这里的store.dispatch相当与reduce的initialValue, 也就是最终目标函数
const composedFn = compose(...chains)
composedFn({action: 'SIMULATE', payload: 1})

// 输出结果
// fn1进, do something...
// fn2进, do something...
// fn3进, do something...
// fn3 getState { name: 'Huaite', age: '25' }
// fn3 dispatch dispatch(action) {
//    console.log('set action to redux, action is', action)
//  }
// set action to redux, action is { action: 'SIMULATE', payload: 1 }
// fn3出, do something...
// fn2出, do something...
// fn1出, do something...
```
### 要点
使用函数的柯里化实现引用在不同的地方传递，使用reduce来封装这个递归的过程
## Share
前端性能优化的一个范例, 非常值得学习和思考
[首屏时间从12.67s到1.06s，我是如何做到的？](https://mp.weixin.qq.com/s/UgU7xrj3e13Jdao7ppWkyA)
