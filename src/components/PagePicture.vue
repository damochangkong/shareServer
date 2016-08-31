<template>
    <div id="singlePic" class="bigImgBox singlePic a">
        <ul>
            <li class="pictureLi" v-for="singlePicture in picArray">
                <img :src="singlePicture.url">
                <template v-for="note in singlePicture.notes">
                    <div v-bind:style="{ left:note.locationx, top:note.locationy }" v-if="note.type == 'text' && note.left" class="tipText">
                        <div v-on:click="magnifyText(note.content)">
                            {{ (note.content.length>8)? (note.content.substring(0,6)+ "……"):note.content}}
                        </div>
                    </div>
                    <div v-bind:style="{ left:note.locationx, top:note.locationy }" v-if="note.type == 'text' && !note.left" class="tipText2">
                        <div v-on:click="magnifyText(note.content)">
                            {{ (note.content.length>8)? (note.content.substring(0,6)+ "……"):note.content}}
                        </div>
                    </div>

                    <div v-bind:style="{ left:note.locationx, top:note.locationy }" v-if="note.type == 'pic' && note.left" class="tipPicture">
                        <div v-on:click="magnifyPic(note.url)">
                            <img :src="note.url"></img>
                        </div>
                    </div>
                    <div v-bind:style="{ left:note.locationx, top:note.locationy }" v-if="note.type == 'pic' && !note.left" class="tipPicture2">
                        <div v-on:click="magnifyPic(note.url)">
                            <img :src="note.url"></img>
                        </div>
                    </div>

                    <div v-bind:style="{ left:note.locationx, top:note.locationy }" v-if="note.type == 'voice' && note.left" class="tipVoice tipVoice1"
                        v-on:click="playVoice($event,note.url)">
                        <div class="voiceOn voiceOn1"></div>
                        <div class="voiceLength voiceLength1">{{note.length}}''</div>
                    </div>
                    <div v-bind:style="{ left:note.locationx, top:note.locationy }" v-if="note.type == 'voice' && !note.left" class="tipVoice tipVoice2"
                        v-on:click="playVoice($event,note.url)">
                        <div class="voiceOn voiceOn2"></div>
                        <div class="voiceLength voiceLength2">{{note.length}}''</div>
                    </div>
                </template>
            </li>
        </ul>
    </div>
    <div id="allNext">
        <div id="allNextBack"></div>
        <div id="backLink" >
            <a v-on:click="backToArticle()"><img src="./images/back.png"/></a>
        </div>
        <div id="indexTip" >
            <span>{{indexTip}}</span>
        </div>
    </div>
    <div id="wholePicBackground" v-show="magnifyPictureShow || magnifyTextShow" class="wholePicBackground">
    </div>
    <div id="centerPicBackground" v-show="magnifyPictureShow || magnifyTextShow" class="centerPicBackground">
        <div class="closePic">
            <div class="closeButton" v-on:click="backToPicture()"><img src="./images/closePic.png"></img></div>
        </div>
        <div class="detailPic" v-show="magnifyPictureShow">
            <img :src="magnifyPictureUrl"></img>
        </div>
        <div class="detailPic detailText" v-show="magnifyTextShow">
            {{magnifyTextContent}}
        </div>
    </div>

</template>
<script>
import $ from 'jquery'
export default {
    data () {
        return {
            picArray: "",
            currentPic : "",
            magnifyPictureShow:false,
            magnifyTextShow:false,
            magnifyPictureUrl:"",
            magnifyTextContent:"",
            indexTip : "",
            scrollImg : {},
            hasNotBindEvent: true
        }
    },
    //这里，即父类可以调用到子类的方法对象：methods,进而调用methods中的成员方法
    methods: {
        showPictureArray: function(_currentPic,_picArray){
            //设置整个数组
            let _initCurrentPic = this.currentPic;
            this.picArray = _picArray;
            this.currentPic = _currentPic;
            var index = this.findIndexInArray(_picArray,_currentPic);
            if((typeof _initCurrentPic)=="object"){
                this.initBigPicture(index,_picArray.length);
            }else{
                setTimeout(this._initBigPicture(index,_picArray.length),10);
            }
        },
        backToArticle: function(){
            //暂停播放
            var audio = document.getElementById("audio");
            if(!audio.paused && ($(".voiceShow").length > 0 || $(".voiceShow2").length > 0)){
                audio.pause();
                $(".voiceOn").removeClass("voiceShow").removeClass("voiceShow2");
            }
            this.$parent.closePictureArray();
        },
        magnifyPic: function(_url){
            this.magnifyPictureShow = true;
            this.magnifyPictureUrl = _url;
        },
        magnifyText: function(_content){
            this.magnifyTextShow = true;
            this.magnifyTextContent = _content;
        },
        playVoice : function(_event,_url){
            //拿到添加样式的div
            var $target = $(_event.srcElement);
            if($(_event.srcElement).hasClass("tipVoice")){
                $target = $(_event.srcElement).find(".voiceOn");
            }
            if($(_event.srcElement).hasClass("voiceLength")){
                $target = $(_event.srcElement).siblings().filter(".voiceOn");
            }
            var audio = document.getElementById("audio");
            if(audio.loop){
                audio.loop = false;
            }
            console.log(audio);
            if($target.hasClass("voiceShow") || $target.hasClass("voiceShow2")){
                //暂停
                $target.removeClass("voiceShow").removeClass("voiceShow2");
                audio.pause();
            }else{
                //播放
                if(audio.src != _url){
                    $(".voiceOn").removeClass("voiceShow").removeClass("voiceShow2");
                    audio.src = _url;
                }
                if($target.parent().hasClass("tipVoice1")){
                    $target.addClass("voiceShow");
                }else{
                    $target.addClass("voiceShow2");
                }
                audio.play();
            }
            $(".voiceDiv .play").show().siblings().hide();
        },
        backToPicture: function(){
            this.magnifyPictureShow = false;
            this.magnifyTextShow = false;
        },
        //此方法为了给initBigPicture传参用的，没有业务意义
        _initBigPicture: function(_index,_arrayLength){
            let _self = this;
            return function(){
                _self.initBigPicture(_index,_arrayLength);
            }
        },
        initBigPicture: function(_index,_arrayLength){
            let _self = this;
            this.scrollImg = $.mggScrollImg('.bigImgBox ul',{
                loop : false,
                auto : false,
                auto_wait_time : 60000,
                firstIndex: _index,
                callback : function(index){
                    index = index==-1?0:index;
                    _self.indexTip = (index+1)+"/"+_arrayLength;
                    if(_self.hasNotBindEvent){
                        console.log("hasNotBindEvent");
                        _self.hasNotBindEvent = false;
                    }
                }
            });
        },
        findIndexInArray: function(targetArray,targetObject){
            let id = targetObject.id;
            for(let i=0;i<targetArray.length;i++){
                if(targetArray[i].id == id){
                    return i;
                }
            }
        }
    }
}
</script>

<style scoped>
* {
    margin:0;
    padding:0;
}
img{
    width: 100%;
    height: 100%;
}
.singlePic{
    position: absolute;
    z-index: 102;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}
.singlePic img{
    width: 100%;
    height: 100%;
}
.singlePic div{
    position: absolute; z-index: 200;
}
.singlePic .tipText{
    width:10rem;
    height:2rem;
    text-align: left;
    background-image:url(./images/tipText.png);
    background-size:cover;
    border-radius:0.3rem;
}
.singlePic .tipText div{
    width:80%;
    height:50%;
    left:20%;
    top:0;
    bottom:0;
    margin:auto;
}
.singlePic .tipText2{
    width:10rem;
    height:2rem;
    text-align: left;
    background-image:url(./images/tipText2.png);
    background-size:100% 100%;
    border-radius:0.3rem;
}
.singlePic .tipText2 div{
    width:80%;
    height:50%;
    left:5%;
    top:0;
    bottom:0;
    margin:auto;
}
.singlePic .tipPicture{
    background-image:url(./images/tipPicture.png);
    background-size:cover;
    width:7.5rem;
    height:4.5rem;
}
.singlePic .tipPicture div{
    width:58%;
    height:90%;
    left:38%;
    margin:3% auto;
    border-radius:0.5rem;
    overflow:hidden;
}
.singlePic .tipPicture2{
    background-image:url(./images/tipPicture2.png);
    background-size:cover;
    width:7.5rem;
    height:4.5rem;
}
.singlePic .tipPicture2 div{
    width:58%;
    height:90%;
    left:3%;
    margin:3% auto;
    border-radius:0.5rem;
    overflow:hidden;
}
.singlePic .tipVoice{
    width:7.5rem;
    height:2.5rem;
    opacity: 1;
    border-radius:0.5rem;
}
.singlePic .tipVoice1{
    background-image:url(./images/tipVoice.png);
    background-size:cover;
}
.singlePic .tipVoice2{
    background-image:url(./images/tipVoice2.png);
    background-size:cover;
}
/*定义语音流动的动画*/
@-webkit-keyframes changeBackground
{
    0% {background-image:url(./images/voice1.png);}
    40% {background-image:url(./images/voice2.png);}
    75% {background-image:url(./images/voice.png);}
}
@-webkit-keyframes changeBackground2
{
    0% {background-image:url(./images/voice21.png);}
    40% {background-image:url(./images/voice22.png);}
    75% {background-image:url(./images/voice20.png);}
}
.voiceShow{
    -webkit-animation: changeBackground 1s infinite;
}
.voiceShow2{
    -webkit-animation: changeBackground2 1s infinite;
}
.voiceOn{
    top:24%;
    width:1em;
    height:1.3em;
}
.voiceOn1{
    left:35%;
    background-image:url(./images/voice.png);
    background-size:cover;
}
.voiceOn2{
    right:35%;
    background-image:url(./images/voice20.png);
    background-size:cover;
}
.voiceLength{
    top:0;
    bottom:0;
    margin:auto;
    color:#ffffff;
    height:1.1rem;
}
.voiceLength1{
    right:5%;
}
.voiceLength2{
    left:5%;
}
#allNext{
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 10%;
    z-index: 102;
}
#allNextBack{
    background-color:#000000;
    width: 100%;
    height: 100%;
    opacity: 0.2;
}
#backLink{
    position: absolute;
    z-index: 103;
    height:3em;
    width:3em;
    left:5%;
    top:0;
    bottom:0;
    margin:auto;
}
#indexTip{
    position: absolute;
    z-index: 103;
    height:2em;
    width:4em;
    font-size:1em;
    left:45%;
    top:0;
    bottom:0;
    margin:auto;
}
.wholePicBackground{
    position: absolute;
    z-index: 300;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    opacity: 0.3;
    background-color:#000000;
}
.centerPicBackground{
    position: absolute;
    z-index: 301;
    top:0;
    bottom:0;
    margin:auto;
    overflow:auto;
    left:3%;
    width: 94%;
    height: 62%;
    background-color:#ffffff;
}
.closePic{
    position: absolute;
    z-index: 302;
    width: 90%;
    height: 15%;
    top:0;
    margin:auto 5%;
}
.closeButton{
    position: absolute;
    z-index: 303;
    height:2em;
    width:2em;
    top:0;
    bottom:0;
    margin:auto;
    right:0%;
    overflow:auto;
}
.detailPic{
    position: absolute;
    z-index: 302;
    top:15%;
    width: 90%;
    height: 80%;
    margin:0 5%;
    border-radius:10px;
}
.detailText{
    font-size:1.5rem;
    text-indent:3rem;
    overflow:scroll;
}
/*控制图片滚动样式*/
ul,li{margin:0;padding:0;}
div.bigImgBox{
    position: absolute;
    overflow:hidden;
    margin:0 auto;
    width:100%;
    height:100%;
}
div.bigImgBox ul{
    clear:both;
    width:2940px;
    height:100%;
}
div.bigImgBox ul li{
    position: relative;
    float:left;
    width:980px;
    height:100%;
    overflow:hidden;
    text-align:center;
}
div.bigImgBox ul li img{
    width:100%;
    height:100%;
}
</style>
