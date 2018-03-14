一个vue的音乐播放器插件，包含播放，切换，显示播放进度，播放实现等功能。
![](https://raw.githubusercontent.com/1053061407/content-manage-front-end/master/src/assets/%E7%BB%84%E4%BB%B6.gif)

由于自己近期在做一个音乐类的桌面端软件，但是在网上，github上也没有找到令自己满意的一些播放器小插件，比如播放暂停按钮，可拖拽进度条等。所以自己就干脆动手做一个。主要记录一下做这个组件的过程，方便以后回忆。

## 1.引入iconfont图标
 把iconfont上的图标项目下载之后，把文件夹下的iconfont.js文件放入自己项目的静态文件夹。
![](http://upload-images.jianshu.io/upload_images/3185709-f7c59c34911a478a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后在`main.js`文件中引入该文件
```
import './assets/js/iconfont'
```
然后在App.vue下全局设置图标的样式
```
<style type="text/css">
  .icon {
    width: 2.5em; height: 2.5em;
    vertical-align: -0.15em;
    fill: currentColor;
    overflow: hidden;
  }
</style>
```
之后就可以在项目中使用了。
例如：
```
<svg class="icon" aria-hidden="true">
   <use xlink:href="#icon-yduishangyiqu"></use>
</svg>
```
这次这个vue的音乐播放器插件按钮主要用的就是这些图标。然后再对图标设置一些v-on:click函数，就大致可以实现音乐的暂停和播放。
## 2. 实现播放暂停功能
因为这个组件里面包含一个<audio :src=musicUrl></audio>元素,所以只需要在子组件中设置一个用于父子组件之间通信的props: { musicUrl: ' '},便可以从父组件中获取音乐链接传给子组件即可。这点并很容易，但是我要实现在父组件中点击歌名，这个插件就会自动播放，并且可以暂停，重新播放就有点难了。一开始我也是设置一个

一开始我是用一个props：{status：' '} 来进行父子组件之间的通信的
```
<controlBtn :musicUrl="musicUrl" :status="status" :songName="songName" :singer="singer">
</controlBtn> 
```
当父组件点击歌名时，在父组件中设置this.status='play'，就可以把'play'传给这个组件的props的status了。
```
props: {
      musicUrl: '',
      status: '',
      singer: '',
      songName: ''
    },
```
但是之后我在这个插件内部设置播放，暂停的状态，还是在props里的status上设置的，比如暂停：this.status='pause'，播放：this.status='play'，但是一直报错，后来意识到vue中不允许子组件中修改用于父子组件通信的props里的值，只能用于父组件传值。后来我就用了官方文档的一个方法，在插件的data中用一个变量来保存这个props里的status的值，然后在这个变量上进行操作。
```
data() {
      return {
        musicStatus: this.status,
      }
 },
```
但是他不是响应式的啊，你props里的status的值改变了，data里的musicStatus并不会改变，他只是一开始初始化的时候赋个值而已。然后我就又想到了用vue的watch属性：
```
watch: {
      status(val) {
        this.musicStatus = val;//新增对prop的status的watch，监听变更并同步到musicStatus上
      }
    },
```
**这样就可以实现data里的musicStatus与props里的status的值同步了。
最后实现播放暂停，也就是通过js代码来控制audio，然后切换musicStatus的值来切换播放和暂停按钮。关于js代码来控制audio，可以看[这篇文章](http://blog.csdn.net/u014520745/article/details/52412427)**    
## 3.实现使用进度条显示播放进度的功能
一开始我想在网上直接找个进度条算了，但是element-ui的进度条没有拖拽的功能，而我们实际听歌的时候，可以拖着进度条的按钮前进或者后退几秒。找了找网上其他的，有个用jquery写的，但是我项目又不想引入jquery，这样把项目弄得很乱（vue项目就用vue写嘛）然后我就决定自己做一个算了。
一开始我的想法就是用HTML5的新增的元素<canvas> 画布来画进度条，毕竟我也看过一些canvas的用法，也在项目中用过。想法大概是用canvas画一条线表示进度条，然后再画一个半径很小的圆，作为进度条上的按钮，通过获取歌曲当前播放的时长，除以歌曲总时长，然后再乘以这个进度条的宽度。但是主要的问题是要让这个圆动起来，让它可以随着音乐播放的进度一直向前移动。然后当时在学习[这个项目](https://github.com/1053061407/canvas-nest)的时候，有了解到`window.requestAnimationFrame() `这个函数，函数的参数是一个回调函数，回调的次数常是每秒60次。所以人眼一般看不出间断，就看起来像个动画，很连贯。利用这个函数，可以使圆随着播放时间变化。
```
status(val) {
        this.musicStatus = val;//新增对prop的status的watch，监听变更并同步到musicStatus上
        if(this.musicStatus == 'play') {
          this.move()
        }
      }
```
首先，当音乐开始播放的时候，开始执行move()函数。move函数的主要部分:
```
var moveLengthX = currentTime*(width-50)/duration。  
//currentTime是当前已经播放的时长，duration是歌曲总时长。
var x = moveLengthX + 50  // 圆心的横坐标，这样圆的横坐标会不断变大
this.drawCircle(ctx,x,20)  //画圆
setTimeout(this.move,10)   
```
**其中setTimeout(this.move,10) 等同于requestAnimationFrame(this.move)，相当于每秒60次的频率执行this.move()函数，反复画圆，这样就画出了歌曲播放进度条。就是开头那个动图的效果啦**

最后贴一下自己的音乐播放器的[地址](https://github.com/1053061407/music-player).目前还在持续开发中。