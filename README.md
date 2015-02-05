演示地址(只适配了chrome)：[http://colorful.coding.io](http://colorful.coding.io)  
原图：

![http://7u2mpp.com1.z0.glb.clouddn.com/colorful.gif](http://7u2mpp.com1.z0.glb.clouddn.com/colorful.gif)

# CSS3 圆自旋动画

> 不要吐嘈，真不知怎么起名好:sweat_smile:

## 前言
再来一次，又是网上见到一个动画，挺漂亮的，接着折腾的思维就冒出来了。自己可不可以把它给画一遍呢，canvas?还是css3?感觉css3是个很大的挑战(当然canvas也是，都还没敢碰)

就这样，又开始了新一轮的折腾之旅。
## 分析动画及开工
一开始的思路呢，当然就是先把小圆给搭起来啦，还好有之前做[这个](http://yedeying999.github.io/2014/09/02/css3-yuan-nei-huan-dong-hua/)的经验。

截图算了一下，小圆个数是24，那也就是每四分之一个大圆就有六个小圆，那好，定个大圆半径，定个小圆半径，就开工了。(所以就是此该的理所当然才导致了之后的重新弄过:sob:)

先搭个html
``` html
<!-- emmet: .board>.circle*24+.center -->
<!DOCTYPE html>
<html lang="en">
<head>
  ...
</head>
<body>
  <div class="board">
    <div class="circle"></div>
    <div class="circle"></div>
    ...
    <div class="circle"></div>
    <div class="center"></div> <!-- 这个是我自己弄的用来定位圆心的小东西 -->
  </div>
</body>
</html>
```
css生成器上，熟悉scss，然后就折腾成这样了
``` scss
$num: 24; // 个数，固定24，乱写后果自负
$width: 400px; // 整个活动区域大小
$outer-radius: 140px; // 大圆半径
$inner-radius: 50px; // 旋转半径
$circle-radius: 20px; // 小圆半径
$center-xpos: $width / 2; // 中心点x
$center-ypos: $width / 2; // 中心点y
$base-xpos: $center-xpos - $circle-radius / 2;
$base-ypos: $center-ypos - $circle-radius / 2;
// 绝对居中定位
%abs-center {
  position: absolute;
  margin: auto;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
}
.board {
  @extend %abs-center;
  width: $width;
  height: $width;
  border: 1px solid #ccc;
}
.center {
  position: absolute;
  top: $center-ypos;
  left: $center-xpos;
  width: 1px;
  height: 1px;
  background-color: #000;
  z-index: 1;
}
// for生成每个小圆的位置
@for $i from 1 through $num {
  $no: $i - 1;
  $angle: (360 / $num * ($no + 0.5)) % 360;
  .circle:nth-child(#{$i}) {
    position: absolute;
    top: $base-ypos + $outer-radius * sin($angle * pi() / 180);
    left: $base-xpos + $outer-radius * cos($angle * pi() / 180);
    width: $circle-radius;
    height: $circle-radius;
    border-radius: 50%;
    // 颜色用色相递增法实现渐变，前几天刚学到的嘻嘻
    background-color: hsl(($angle + 180) % 360, 100%, 50%);
  }
}
```

结果出来了，挺漂亮，棒棒哒。至此，代码为circle分支

![first version](http://7u2mpp.com1.z0.glb.clouddn.com/pic1.jpg)

## 接着，死了
开始研究它的动作，然后研究了半天，结果发现自己天真了:scream:

它的动作分为以下几点：

* 每个小圆和它的下三个或上三个是同一组，中间隔了两个，每个圆只属于一组
* 同一组的两个圆会在轮到它们转时绕两点之间中心旋转，且时针方向一致，为顺时针
* 同一组的圆在旋转时，其对应颜色会相互过渡成对方的

现在问题来了，CSS3可以旋转，可以我现在以谁为圆心？没有！好吧，要改html结构了。

深思熟虑一会后，决定给每组的圆中间加条棍，然后再让棍来旋转就好，所以html变成了这样：
``` html
<!-- emmet: .board>(.box*12>.circle*2)+.center -->
<!DOCTYPE html>
<html lang="en">
<head>
  ...
</head>
<body>
  <div class="board">
    <div class="box">
      <div class="circle"></div>
      <div class="circle"></div>
    </div>
    ...
    <div class="box">
      <div class="circle"></div>
      <div class="circle"></div>
    </div>
    <div class="center"></div>
  </div>
</body>
</html>
```
接下来css就难做了，主要是尺寸问题。
### 尺寸计算
刚刚的小圆布局已经要扔掉了，不过有些尺寸变量还是可以留着的。

木棍的个数是小圆个数的1/2，然后旋转角度也按12份平分一个圆，宽度多少没问题，我选择容纳一个圆的直径的宽度，另外还有木棍中点到原点距离以及木棍长度。木棍长度可以把两小圆圆心连一条线到原点，这个度数应该是`45deg`(刚开始不知道为什么脑抽了算成`60`，然后，错了好久:sob:)，这样可以算出这两个值。

分析完了，看代码：
``` scss
$num: 24;
$width: 400px;
$outer-radius: 140px;
$inner-radius: 50px;
$circle-radius: 20px;
$box-num: $num / 2;
$center-xpos: $width / 2;
$center-ypos: $width / 2;
$box-width: $circle-radius;
$box-height: $outer-radius;
$base-xpos: $center-xpos - $circle-radius / 2;
$base-ypos: $center-ypos - $circle-radius / 2;
%abs-center {
  position: absolute;
  margin: auto;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
}
.board {
  @extend %abs-center;
  width: $width;
  height: $width;
  border: 1px solid #ccc;
}
.center {
  position: absolute;
  top: $center-ypos;
  left: $center-xpos;
  width: 1px;
  height: 1px;
  background-color: #000;
  z-index: 1;
}
.circle:first-child {
  position: absolute;
  width: $circle-radius;
  height: $circle-radius;
  border-radius: 50%;
  left: 0;
  top: 0 - $circle-radius / 2;
  // background-color: red;
}
.circle:last-child {
  position: absolute;
  width: $circle-radius;
  height: $circle-radius;
  border-radius: 50%;
  left: 0;
  bottom: 0 - $circle-radius / 2;
  // background-color: blue;
}
@for $i from 0 to $num {
  $angle: (360 / $num * ($i + 0.5)) % 360;
  $offset-angle: $angle + (360 / $num * 3);
  @-webkit-keyframes rotate#{$i} {
    0% {
      top: $base-ypos + $outer-radius * sin($angle * pi() / 180);
      left: $base-xpos + $outer-radius * cos($angle * pi() / 180);
    } 16.666667%, 83.333333% {
      top: $base-ypos + $outer-radius * sin($offset-angle * pi() / 180);
      left: $base-xpos + $outer-radius * cos($offset-angle * pi() / 180);
    } 100% {
      top: $base-ypos + $outer-radius * sin($angle * pi() / 180);
      left: $base-xpos + $outer-radius * cos($angle * pi() / 180);
    }
  }
}
@for $i from 1 through $box-num {
  $no: $i - 1;
  $angle: 360 / $box-num * $no;
  .box:nth-child(#{$i}) {
    position: absolute;
    background-color: rgba(0,0,0,.3);
    left: $center-xpos + $outer-radius * cos($angle * pi() / 180) - $box-width / 2;
    top: $center-ypos + $outer-radius * sin($angle * pi() / 180) - $box-height / 2;
    width: $box-width;
    height: $box-height;
    -webkit-transform: rotate(($angle) % 360 + deg);
  }
}
```
然后这是木棍的图，挺漂亮的有木有。至此，代码为rect分支

![rect](http://7u2mpp.com1.z0.glb.clouddn.com/pic2.jpg)

## 添加动画

动画有几个值得一提的点：

* 木棍是轮流旋转的，animation-delay要控制为 `总时间 * 当前木棍序号 / 总木棍序号`
* 关键帧要注意，经观察，转动时间于不转时间大概为 `1:2`，因此定了`0----1/6====1/2----2/3====1`作为关键关键帧(-表动=表静)
* 需要变换的属性有木棍旋转角度、小圆颜色、小圆旋转轨迹
* 因小圆旋转轨迹是个圆，干脆把木棍拉宽，然后它就成了一个圆饼木棍哈哈，以其边框作为轨迹
* timing-function需要调一下，我调得不是很好，也和原图差得比较大

不废话了，如下：
``` scss
$num: 24;
$width: 400px;
$outer-radius: 140px;
$inner-radius: 50px;
$circle-radius: 20px;
$box-num: $num / 2;
$center-xpos: $width / 2;
$center-ypos: $width / 2;
$box-width: $outer-radius * sin(pi() * 3 / $num) * 2;
$box-height: $box-width;
$box-radius: $outer-radius * cos(pi() * 3 / $num);
$base-xpos: $center-xpos - $circle-radius / 2;
$base-ypos: $center-ypos - $circle-radius / 2;
%abs-center {
  position: absolute;
  margin: auto;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
}
.origin {
  text-align: center;
  margin-top: 20px;
}
.board {
  @extend %abs-center;
  width: $width;
  height: $width;
  border: 1px solid #ccc;
}
.center {
  position: absolute;
  top: $center-ypos;
  left: $center-xpos;
  width: 1px;
  height: 1px;
  background-color: #000;
  z-index: 1;
}
.box {
  position: absolute;
  width: $box-width;
  height: $box-height;
  border-radius: 50%;
  border-width: 1px;
  border-style: solid;
  -webkit-animation-duration: 10s;
  -webkit-animation-iteration-count: infinite;
  -webkit-animation-timing-function: cubic-bezier(0.5, 0.0, 0.5, 1.0);
}
.circle {
  position: absolute;
  width: $circle-radius;
  height: $circle-radius;
  border-radius: 50%;
  left: $box-width / 2 - $circle-radius / 2;
  -webkit-animation-duration: 10s;
  -webkit-animation-iteration-count: infinite;
  -webkit-animation-timing-function: cubic-bezier(0.5, 0.0, 0.5, 1.0);
  &:first-child {
    top: -$circle-radius / 2;
  }
  &:last-child {
    bottom: -$circle-radius / 2;
  }
}
@for $i from 0 to $box-num {
  $angle: 360 / $box-num * $i;
  @-webkit-keyframes rotate#{$i} {
    0% {
      -webkit-transform: rotate(($angle) % 360 + deg);
      border-color: rgba(hsl(($angle + 202.5) % 360, 100%, 50%), 0);
    } 8.333333% {
      border-color: rgba(hsl(($angle + 202.5) % 360, 100%, 50%), .7);
    } 16.666667%, 50% {
      border-color: rgba(hsl(($angle + 202.5) % 360, 100%, 50%), 0);
      -webkit-transform: rotate(($angle + 180) + deg);
    } 58.333333% {
      border-color: rgba(hsl(($angle + 202.5) % 360, 100%, 50%), .7);
    } 66.666667%, 100% { 
      border-color: rgba(hsl(($angle + 202.5) % 360, 100%, 50%), 0);
      -webkit-transform: rotate(($angle + 360) + deg);
    }
  }
  @-webkit-keyframes colorfir#{$i} {
    0% {
      background-color: hsl(($angle + 180) % 360, 100%, 50%);
    } 16.666667%, 50% {
      background-color: hsl(($angle + 225) % 360, 100%, 50%);
    } 66.666667%, 100% {
      background-color: hsl(($angle + 180) % 360, 100%, 50%);
    }
  }
  @-webkit-keyframes colorlas#{$i} {
    0% {
      background-color: hsl(($angle + 225) % 360, 100%, 50%);
    } 16.666667%, 50% {
      background-color: hsl(($angle + 180) % 360, 100%, 50%);
    } 66.666667%, 100% {
      background-color: hsl(($angle + 225) % 360, 100%, 50%);
    }
  }
}
@for $i from 1 through $box-num {
  $no: $i - 1;
  $angle: 360 / $box-num * $no;
  .box:nth-child(#{$i}) {
    left: $center-xpos + $box-radius * cos($angle * pi() / 180) - $box-width / 2;
    top: $center-ypos + $box-radius * sin($angle * pi() / 180) - $box-height / 2;
    border-color: rgba(hsl(($angle + 202.5) % 360, 100%, 50%), 0);
    -webkit-transform: rotate(($angle) % 360 + deg);
    -webkit-animation-name: rotate#{$no};
    -webkit-animation-delay: $no * 10.0 / 24 + s;
    .circle:first-child {
      background-color: hsl(($angle + 180) % 360, 100%, 50%);
      -webkit-animation-name: colorfir#{$no};
      -webkit-animation-delay: $no * 10.0 / 24 + s;
    }
    .circle:last-child {
      background-color: hsl(($angle + 225) % 360, 100%, 50%);
      -webkit-animation-name: colorlas#{$no};
      -webkit-animation-delay: $no * 10.0 / 24 + s;
    }
  }
}
```
至此，文章一开始的那个在线演示就出来啦，再加个fork装装逼，完满结束。分享下过程图：

![过程图](http://7u2mpp.com1.z0.glb.clouddn.com/pic3.jpg)![过程图](http://7u2mpp.com1.z0.glb.clouddn.com/pic4.jpg)