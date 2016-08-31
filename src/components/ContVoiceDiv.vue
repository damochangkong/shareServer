<template>
    <div id="voiceCont" class="voiceCont a">
        <audio id="audio" src="" loop="loop"></audio>
        <div class="voiceNotes" id="voiceNotes">
            <ul>
                <template v-for="item in noteList">
                    <li class="note borderLing">
                        <div class="noteDiv">
                            <div class="timeTip" v-on:click="playMyTime(item.time,$index)">
                                <div class="playButton"><img src="./images/playButton.png"></img></div>
                                <span>{{ item.time }}</span>
                                <div class="circle"><img src="./images/circle.png"></img></div>
                            </div>
                            <div class="wordTip" v-on:click="showWholeContent($event,item.content)">
                                <p>{{ (item.content.length>18)? (item.content.substring(0,16)+ "……"):item.content }}</p>
                            </div>
                        </div>
                    </li>
                 </template>
            </ul>
        </div>
        <!--时间进度条块儿start-->
        <section class="progressBar">
            <div class="progressBac"></div>
            <div class="speed" id="speed"></div>
            <div class="drag" id="drag"></div>
            <!--播放控制按钮start-->
            <button id="control" class=""></button>
        </section>
        <!--播放时间start-->
        <div id="time">
            <div class="timeDetail">
                <span class="currentTime" id="currentTime">00:00</span>/<span class="allTime" id="allTime">00:00</span>
            </div>
        </div>
    </div>
</template>
<script>
import $ from 'jquery'
export default {
    data () {
        return {
            startX:'', //触摸时的坐标
            x:'', //滑动的距离
            aboveX: 0, //设一个全局变量记录上一次内部块滑动的位置
            noteList:[],
            preContDiv:""
        }
    },
    //这里，即父类可以调用到子类的方法对象：methods,进而调用methods中的成员方法
    methods: {
        //获取歌曲链接并插入dom中
        getSong: function(_item) {
            var audio = document.getElementById("audio");
            if(audio.src != _item.url){
                audio.src = _item.url;
                audio.loop = false; //歌曲循环
                this.noteList = _item.notes;
                this.playControl(); //播放控制函数
                audio.play();
            }
        },
        //点击播放/暂停
        clicks: function () {
            var _self = this;
            var audio = document.getElementById("audio");
            $("#control").click(function() {
                //播放
                console.log($("#control"));
                if ($("#control").hasClass("play")) {
                    $("#control").addClass("pause").removeClass("play");
                    _self.$parent.$children[4].playingVoiceDiv.find(".pause").show().siblings().hide();
                    audio.play();//开始播放
                    _self.dragMove();//并且滚动条开始滑动

                } else {
                    //暂停
                    $("#control").addClass("play").removeClass("pause");
                    _self.$parent.$children[4].playingVoiceDiv.find(".play").show().siblings().hide();

                    audio.pause();
                }
            })
        },
        //播放时间  默认获取的时间是时间戳改成我们常见的时间格式
        timeChange : function (time, timePlace) {
            var timePlace = document.getElementById(timePlace);
            //分钟
            var minute = time / 60;
            var minutes = parseInt(minute);
            if (minutes < 10) {
                minutes = "0" + minutes;
            }
            //秒
            var second = time % 60;
            var seconds = parseInt(second);
            if (seconds < 10) {
                seconds = "0" + seconds;
            }
            var allTime = "" + minutes + "" + ":" + "" + seconds + ""
            timePlace.innerHTML = allTime;
        },
        //播放事件监听
        playControl : function () {
            let _self = this;
            //歌曲一经完整的加载完毕( 也可以写成上面提到的那些事件类型)
            audio.addEventListener("loadeddata", function() {
                _self.addListenTouch(); //歌曲加载之后才可以拖动进度条
                var allTime = audio.duration;
                _self.timeChange(allTime, "allTime");
                setInterval(function() {
                    var currentTime = audio.currentTime;
                    $("#time .currentTime").html(
                        _self.timeChange(currentTime, "currentTime"));
                }, 1000);
                _self.clicks();
            }, false);
            //监听暂停
            audio.addEventListener("pause", function() {
                $("#control").addClass("play").removeClass("pause");

                if (audio.currentTime == audio.duration) {
                    audio.currentTime = 0;
                }
            }, false);
            //监听播放
            audio.addEventListener("play",function() {
                $("#control").addClass("pause").removeClass("play");

                _self.dragMove();
            }, false);
            //监听
            audio.addEventListener("ended", function() {
                var audio = document.getElementById("audio");
                audio.currentTime = 0;
                $("#control").addClass("play").removeClass("pause");
                _self.$parent.$children[4].playingVoiceDiv.find(".play").show().siblings().hide();
                $(".voiceOn").removeClass("voiceShow").removeClass("voiceShow2");

            }, false)
        },
        //进度条 这里用的是事件实现进度拖动
        //拖动监听touch事件
        addListenTouch : function () {
            document.getElementById("drag").addEventListener("touchstart",
                this.touchStart, false);
            document.getElementById("drag").addEventListener("touchmove",
                this.touchMove, false);
            document.getElementById("drag").addEventListener("touchend",
                this.touchEnd, false);
            var drag = document.getElementById("drag");
            var speed = document.getElementById("speed");
        },
        //touchstart,touchmove,touchend事件函数
        touchStart : function (e) {
            e.preventDefault();
            var touch = e.touches[0];
            this.startX = touch.pageX;
        },
        //滑动
        touchMove : function (e) {
            e.preventDefault();
            var touch = e.touches[0];
            this.x = touch.pageX - this.startX; //滑动的距离
            //drag.style.webkitTransform = 'translate(' + 0+ 'px, ' + y + 'px)';  //也可以用css3的方式
            drag.style.left = this.aboveX + this.x + "px"; //
            speed.style.left = -((window.innerWidth) - (this.aboveX + this.x)) + "px";
        },
        //手指离开屏幕
        touchEnd : function (e) {
            e.preventDefault();
            this.aboveX = parseInt(drag.style.left);
            var touch = e.touches[0];
            var dragPaddingLeft = drag.style.left;
            var change = dragPaddingLeft.replace("px", "");
            var numDragpaddingLeft = parseInt(change);
            var currentTime = (numDragpaddingLeft / (window.innerWidth - 30))
                * audio.duration;//30是拖动圆圈的长度，减掉是为了让歌曲结束的时候不会跑到window以外
            audio.currentTime = currentTime;
        },
        //3拖动的滑动条前进
        dragMove : function () {
            setInterval(
                function() {
                    drag.style.left = (audio.currentTime / audio.duration)
                        * (window.innerWidth - 30) + "px";
                    speed.style.left = -((window.innerWidth) - (audio.currentTime / audio.duration)
                        * (window.innerWidth - 30))
                        + "px";
                }, 500);
        },
        playMyTime : function(_time , $index){
            var times = _time.split(":");
            var _currentTime = parseInt(times[0]*60) + parseInt(times[1]);
            var audio = document.getElementById("audio");
            audio.currentTime = _currentTime;
            $(".timeTip").removeClass("selected").eq($index).addClass("selected");
        },
        showWholeContent : function(_event,_content){
            //收起前一个div内容
            console.log(this.preContDiv);
            if((typeof this.preContDiv)=="object"){
                var cont = $(this.preContDiv).text().trim();
                cont = (cont.length>18)?(cont.substring(0,16)+ "……"):cont;
                this.preContDiv.text(cont);
            }
            //展开现在的div内容
            $(_event.srcElement).text(_content);
            this.preContDiv = $(_event.srcElement);
        }
    }
}
</script>
<style scoped>
* {
	margin: 0;
	padding: 0;
}
img{
  width:100%;
  height:100%;
}
.selected{
  border: 1px solid gray;
}
.voice{
  position : absolute;
  width:100%;
  height:100%;
}
.voiceCont{
  position: absolute;
  width:100%;
  height:86%;
  top:14%;
  background:#ffffff;
  text-align:center;
  z-index: 300;
  overflow:scroll;
}
.voiceNotes {
  width:100%;
  height:80%;
  background:#ffffff;
  float:left;
  font-size:1rem;
  overflow:scroll;
}
.voiceNotes ul {
  list-style:none;
  width:100%;
  height:100%;
}
.voiceNotes ul li{
  display:block;
  text-align:left;
  width:97%;
  float:left;
  padding:0.3rem;
  margin-bottom : o.5rem;
  line-height:1rem;
}
.noteDiv{
   width:100%;
   height:100%;
   position:relative;
   font-size:1rem;
}
.timeTip{
  position:relative;
  float:left;
  width:24%;
  height:1rem;
  left:0;
}
.timeTip div{
   float:left;
   width:1rem;
   height:1rem;
   margin-left:0.5rem;
}
.timeTip span{
   float:left;
   margin: 0 0.5rem;
}
.timeTip .circle{
  margin-left:0;
}
.wordTip {
  position:relative;
  float:left;
  width:72%;
  text-align:left;
  margin-left:0.3rem;
}
.wordTip span{
  display:inline;
  text-indent:2rem;
}

#audio {
	width: 100%;
}
#control {
	width: 150px;
	height: 150px;
	line-height: 150px;
	text-align: center;
	border-radius: 200px;
	border: none;
	color: #fff;
	margin-top: -60px;
	margin-left: -75px;
	left: 50%;
	top: 50%;
	position: absolute;
	font-size:1rem;
}
.hide {
	display: none;
}
.show {
	display: block;
}
.play {
  background-image:url(./images/play.png);
  background-size:cover;
}
.pause {
  background-image:url(./images/pause.png);
  background-size:cover;
}
/*进度条样式*/
.progressBar {
	width: 100%;
	height: 20%;
	margin: 30px auto 0px auto;
	position: absolute;
	left: 0;
	bottom: 0px;
	background: #555555;
}
.progressBar div {
	position: absolute;
}
.progressBar .progressBac {
	width: 100%;
	height: 10px;
	left: 0;
	top: 30px;
	background: #999999;
}
.progressBar .speed {
	width: 100%;
	height: 10px;
	left: -100%;
	top: 30px;
	background: #ffffff;
}
.progressBar .drag {
	width: 30px;
	height: 30px;
	left: 0;
	top: 20px;
	background: #ffffff;
	opacity: 0.8;
	border-radius: 50px;
	box-shadow: #fff 0 0 5px;
}
/*时间样式*/
#time {
	width: 100%;
	height: 20px;
	position: absolute;
	left: 0;
	bottom: 30px;
	color: #888;
}
.timeDetail {
	position: absolute;
	right: 10px;
	top: 0;
}
.timeDetail span{
	color:#ffffff;
}
</style>
