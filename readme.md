
> 懒加载是一种网页性能优化的方式，它能极大的提升用户体验。就比如说图片，图片一直是影响网页性能的主要元凶，现在一张图片超过几兆已经是很经常的事>了。... 所以，我们需要懒加载，进入页面的时候，只请求可视区域的图片资源

<!-- more -->

**问题:**

1. 全部加载的话会影响用户体验
2. 浪费用户的流量，有些用户并不像全部看完，全部加载会耗费大量流量。

### html实现
最简单的实现方式是给`img`标签加上 `loading='lazy'`，该属性的兼容性也还行，生成环境可以使用
```html
<img src="img/31.png" loading='lazy'/>
```


### js实现
我们通过js监听页面的滚动也能实现。  
使用js实现的原理主要是判断当前图片是否到了可视区域:  
- 拿到所有的图片dom。
- 遍历每个图片判断当前图片是否到了可视区范围内。
- 如到达就设置图片的sre属性。
- 绑定window的scroll事件，对其进行事件监听。
在页面初始化的时候，图片的src实际上是放在data-src属性
上的，当元素处于可视范围内的时候，就把data-src赋值给
src属性，完成图片加载。
```html
<!-- 初始 -->
<img data-src="img/31.png" src=""/>
<!-- 可视区域 -->
<img data-src="" src="img/31.png"/>
```

**demo 例子**

```html
<!DOCTYPE html>
<html lang="en" />

<head>
  <meta charset="UTF-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>懒加载</title>
  <style>
    img { width: 80vw; height: calc(80vw*160/256); margin-bottom: 150px; display: block; }
    img[src=''] { opacity: 0; }
    h1 { position: sticky; top: 0; }
  </style>
</head>

<body>
  <h1 contenteditable>懒加载 标题可编辑</h1>
  <img data-src="img/31.png" /> <img data-src="img/32.png" /> <img data-src="img/33.png" />
  <img data-src="img/34.png" /> <img data-src="img/35.png" /> <img data-src="img/36.png" />
  <img data-src="img/37.png" /> <img data-src="img/38.png" /> <img data-src="img/39.png" />
  <img data-src="img/40.png" /> <img data-src="img/41.png" /> <img data-src="img/42.png" />
  <img data-src="img/43.png" /> <img data-src="img/44.png" /> <img data-src="img/45.png" />
</body>

</html>
```

先获取所有图片的 dom，通过`document.body.clientHeight`获取可视区高度，再使用`item.getBoundingClientRect()`API 直接得到元素相对浏览的 `top` 值，遍历每个图片判断当前图片是否到了可视区范围内。

```js
 function lazyload () {
   const imgs = document.querySelectorAll('img[data-src]');
   let viewHeight = document.body.clientHeight;

   imgs.forEach(item => {
     if (item.dataset.src === '') return;
    
    //用于获得页面某个元素的左，上，右，下分别相对浏览器视窗的位置
     let rect = item.getBoundingClientRect();
     if (rect.bottom >= 0 && rect.top < viewHeight) {
       item.src = item.dataset.src;
       item.removeAttribute('data-src');
     }
   })
 }
```

最后给 window 绑定 `scroll` 事件

```js
window.addEventListener("scroll", lazyload);
```

主要就完成了一个图片懒加载的操作了。但是这样存在较 大的性能问题，因为`scroll`事件会在很短的时间内触发很多次，严重影响页面性能，为了提高网页性能， 我们需要一个节流函数来控制函数的多次触发，在一段时间内(`200ms`)只执行 1 次回调。  
下面实现一个节流函数

```js
function throttle(fn, delay) {
  let startTime = 0;
  return function () {
    let args = arguments;
    let that = this;
    let curTime = Date.now();
    if (curTime - startTime > delay) {
      fn.apply(that, args);
      startTime = curTime;
    }
  };
}
```

然后修改`scroll`事件

```js
window.addEventListener("scroll", throttle(lazyload, 200));
```

### 扩展

**IntersectionObserver**

通过上面例子，我们要实现懒加载都需要去监听 `scroll`事件，尽管我们可以通过**函数节流**的方式来阻止高频率的执行函数，但是我们还是需要去计算`scrollTop`, `offsetHeight`等属性，有没有简单的不需要计算这些属性的方式呢？就是**IntersectionObserver**。  
`IntersectionObserver`是一个比较新的 API,可以自动"观察"元素是否可见，Chrome51+ 已经支持。我们来看一下它的用法:

```js
const observer = new IntersectionObserver(callback, option);
//开始观察
observer.observe(document.getElementById("example"));
//停止观察
observer.unobserve(element);
//关闭观察器
observer.disconnect();
```

`Ieretitnororor`是浏览器原生提供的构造函数，接受两个参数:**callback** 是可见性变化时的回调函数，**option** 是配置对象(可选) 。  
目标元素的可见性变化时，就会调用观察器的回调函数callback，一般会执行两次，一次是目标元素刚进入规口(开始可见,另次是完全离开视口(开始不可见) 。   
下面用`ItersctionObserver`.实现图片懒加载

```js
 function lazyload () {
   const imgs = document.querySelectorAll('img[data-src]');
   const config = {
     rootMargin: '0px',
     threshold: 0
   }

   let observer = new IntersectionObserver((entries, self) => {
     entries.forEach(item => {
       if (item.isIntersecting) {
         let img = item.target;
         let src = img.dataset.src;
         if (src) {
           img.src = src
           img.removeAttribute('data-src');
         }
         self.unobserve(item.target);
       }
     })
   }, config);

   imgs.forEach(item => observer.observe(item));
 }

 window.addEventListener('load', lazyload);
 window.addEventListener("scroll", throttle(lazyload, 200));
```

[demo效果](lazyload.vercel.app)
