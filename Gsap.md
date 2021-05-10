# 1.Gsap

## Timeline

- 从整体上控制一组动画。
- 在不使用很多`delay`的情况下构建一个动画序列。（如果对前一个动画进行时间调整后一个动画的触发时间也会改变，从而大大简化了实验和维护工作）。
- 对动画进行模块化。
- 可以进行非常复杂的动画编排。
- 要基于一组动画触发回调（例如“在完成所有这些动画之后，调用`myFunction()`”）。

生命时间线变量`var t1 = gsap.timeline();`

然后将需要依次触发的动画添加入时间线里即可，例如：

```js
var tl = gsap.timeline();
tl.add(gsap.to("#app", {
    duration: 1,
    delay: 1,
    x: 500,
}));
tl.to("#app", {
    duration: 1,
    y: 500,
});
```

时间轴的特殊属性：

- `repeat`：动画重复的次数。
- `repeatDelay`：两次重复之间的间隔时间（以秒为单位）。
- `yoyo`：如果为`true`，则每次重复播放都会前后交替进行。
- `delay`：时间轴开始之前的延迟（以秒为单位）。
- `onComplete`：时间线播放完毕后调用的函数。

```js
var tl = gsap.timeline({
  repeat: 1, 
  yoyo: true, 
  onRepeat: onRepeatHandler,
  onComplete: onCompleteHandler
});
```

**Getter / Setter 方法**

- `time()` 播放头的本地位置（当前时间，以秒为单位），不包括任何重复或repeatDelays。
- `progress()` 它是介于0和1之间的值，指示播放头的位置，其中0处在开始位置，0.5处在中途完成，1处在结束位置。
- `duration()` 动画的持续时间（以秒为单位），不包括任何重复或repeatDelays。
- `delay()` 动画的初始延迟（动画开始之前的时间长度，以秒为单位）。

## SplitText

### 1.基本用法

-  首先创建一个新的SplitText实例，然后将以下任何内容传递给构造函数以指示要拆分的元素：DOM元素，DOM元素数组，选择器对象（例如jQuery对象）或选择器文本。例如： 

```js
//a DOM element:
var yourElement = document.getElementById("yourID");
var split = new SplitText(yourElement);
//or a selector object, like jQuery:
var split = new SplitText( $(".yourClass") );
//or selector text which will use jQuery by default (if loaded) to get the selection:
var split = new SplitText("#yourID");
//or an array of DOM elements:
var split = new SplitText([element1, element2, element3]);
```

