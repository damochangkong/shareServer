<template>
    <div class="article add">
        <div class="titleWord borderLing">
            <span class="articleTitle">{{articleTitle}}</span><br/>
            <span class="sectionCount">{{sectionCount}}个段落</span>
        </div>
        <div class="contentArticle borderLing" v-for="item of articleItems">
            <template v-if="item.type == 'text'">
                <div id="textDiv" :class="item.location">
                    <p>{{item.text}}</p>
                </div>
            </template>
            <template v-if="item.type == 'pic'">
                <div id="pictureDiv" class="imgBox">
                    <ul>
                        <li v-for="pictureItem in item.pics">
                            <a v-on:click="showPicture(pictureItem,item.pics)"><img :src="pictureItem.url"></a>
                        </li>
                    </ul>
                    <div class="pageDiv"><span class="currentPage">1</span>/<span class="totalPage">{{item.pics.length}}</span></div>
                </div>
            </template>
            <template v-if="item.type == 'voice'">
                <div :class="item.location" v-on:click="showOrPlayVoice(item,$event)" class="voiceDiv" >
                    <img class="play" src="./images/voicePlay.png"></img>
                    <img class="pause" :id= "item.location" src="./images/voicePause.png" style="display:none"></img>
                </div>
            </template>
        </div>
    </div>
</template>
<script>

import $ from 'jquery'
import mggScrollImg from './js/picSlide'

export default {
     //这里定义本页的成员变量
     data(){
         console.log("contArticleDiv");
         return {
             articleTitle : "",
             articleItems : [],
             navigates:[],
             sectionCount:"",
             playingVoiceDiv:{}
         }
     },
     methods: {
         initArticleData: function(_content){
             console.log("typeof _content:" + typeof _content);
             //分享的是长文
             if((typeof _content) == "object"){
                 this.jsonpCallBack(this,_content,"content");
                 this._setNavigate(true);
             }else{
                 this.requestRemoteArticleData(_content);
                 this._setNavigate(false);
             }
         },
         _setNavigate : function(_value){
             this.$parent.$children[3].showOneNavigate = _value;
         },
         //设置导航目录
         _setNavigateItem : function(_array){
             console.log(_array);
             this.$parent.$children[3].navigates = _array;
         },
         requestRemoteArticleData : function(_data){
             var _self = this;
             $.ajax({
                 type : "POST",
                 url : "http://localhost:8081/shareWeb/doc/articleLong.json",
                 data : {},
                 dataType : "jsonp",
                 jsonp:"callback",
                 async : false,
                 jsonpCallback:"jsonpCallback",
                 success : function(data) {
                     _self.jsonpCallBack(_self,data,"node");
                 },
                 error:function(data){
                      console.log(data);
                 }
             })
         },
         jsonpCallBack : function(_self,data,type){
             //解析长文节点,设置标题等属性
             if(type=="content"){
                 //从目录进来的，设置导航条
                 var username = _self.$root.$children[0].personInfo.name;
                 _self._setNavigateItem([{name:username},{name:data.name}]);
             }
             _self.articleTitle = data.name;
             _self.sectionCount = data.items.length;
             _self.articleItems = data.items;
             setTimeout(_self.initPicture,100);
         },
         initPicture : function(){
             //给图片集绑定滚动逻辑
             $('.imgBox ul').each(function(){
                 var _self = this;
                 $.mggScrollImg(_self,{
                      loop : true,
                      auto : true,
                      auto_wait_time : 6000,
                      callback : function(ind){
                          $(_self).siblings().find(".currentPage").text(ind+1);
                      }
                 });
             })
         },
         showPicture: function(currentPic,picArray){
             this.$parent.$parent.showPictureSlide(currentPic,picArray);
         },
         showOrPlayVoice: function(item,_event){
             var $target = $(_event.srcElement);
             //非图片区域
             if($target.hasClass("voiceDiv")){
                 this.playingVoiceDiv = $target;
                 this.dealStyle(item,$target);
                 this.$parent.showVoicePage(item);
             }else{
                 this.playingVoiceDiv = $target.parent();
                 //如果点击的是图片
                 if(_event.offsetX >= 169 && _event.offsetX <= 318){
                     this.playVoice(item,$target);
                 }else{
                     //图片区域，但是不是中心区域

                     this.dealStyle(item,$target.parent());
                     this.$parent.showVoicePage(item);
                 }
             }
         },
         dealStyle : function(_item,$target){

             var audio = document.getElementById("audio");
             console.log("audio.src:"+audio.src+",_item.url:"+_item.url);
             if(audio.src != _item.url){
                 $(".voiceDiv .play").show().siblings().hide();
                 $target.find(".pause").show().siblings().hide();

             }
         },
         //点击播放或者暂停按钮
         playVoice: function(_item,_imgSrc){
              var audio = document.getElementById("audio");
              if(audio.src != _item.url){
                  //播放，设置成暂停样式
                  if(!audio.paused){
                      $(".voiceDiv .pause").hide().siblings().show();
                  }
                  _imgSrc.hide().siblings().show();
                  this.$parent.playVoiceSilence(_item);
              }else{
                  if(audio.paused){
                      audio.play();
                      //播放，设置成暂停样式
                      _imgSrc.hide().siblings().show();
                  }else{
                      audio.pause();
                      //暂停，设置成播放样式
                      _imgSrc.hide().siblings().show();
                  }
              }
         },
         pauseVoice : function(){
             var audio = document.getElementById("audio");
             if(!audio.paused){
                 audio.pause();
                 $(".voiceDiv").find(".pause").hide().siblings().show();
             }
         },
         backToNoteList : function(_index){
             this.pauseVoice();
             if(_index==0){
                 this.$parent.backToNoteListFromArticle();
             }
         },
         dealArticleVoice : function(){

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
  width:100%;
  height:100%;
}
.article {
  position: absolute;
  width:100%;
  height:86%;
  top:14%;
  background:#ffffff;
  text-align:center;
  z-index: 300;
  overflow:scroll;
}

.titleWord{
  float:left;
  text-align:left;
  width:95%;
  padding:2% 0 2% 5%;
  height:15;
  line-height:2.5rem;
  background:#3e3e50;
  z-index: 301;
}
.titleWord .articleTitle{
  float:left;
  color:#ffffff;
  font-size:1.5rem;
}
.titleWord .sectionCount{
  float:left;
  color:#ffffff;
  font-size:1rem;
}
.contentArticle{
  width:100%;
  margin:1% auto 0 auto;
  z-index: 301;
  float:left;
  overflow : scroll;
}
#textDiv{
  margin:0 5%;
}
#textDiv p{
  text-align:left;
  text-indent:2rem;
  font-size:1em;
}
#pictureDiv{
  text-align:center;
  margin:0 auto;
  z-index: 302;
}
.pageDiv{
  float:left;
  width:100%;
}
.voiceDiv{
  width:100%;
  height:50%;
  float:left;
  z-index: 302;
}
.voiceDiv img{
  width:50%;
  height:80%;
  z-index: 303;
}
ul,li{margin:0;padding:0;}
div.imgBox{
    width:27rem;
    overflow:hidden;
    margin:0 auto;
}
div.imgBox ul{
    clear:both;
    width:81rem;
    height:100%;
}
div.imgBox ul li{
    float:left;
    width:27rem;
    height:19rem;
    overflow:hidden;
    text-align:center;
}
div.imgBox ul li img{
    width:27rem;
    height:19rem;
}
div.imgBox ul li a{
    width:27rem;
    height:19rem;
}
#page{
    color:red;
}

</style>
