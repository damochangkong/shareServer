<template>
    <div id="login" class="login" v-show="loginShow">
        <page-login-div></page-login-div>
    </div>
    <div id="app" class="app" v-show="appShow">
        <page-article></page-article>
    </div>
    <div id="allPic" v-show="pictureShow">
        <page-picture></page-picture>
    </div>
    <div id="loading " v-show="loadingShow dd">
        <div class="loadingContent">正在加载……</div>
    </div>
</template>
<script>
import $ from 'jquery'
import PageLoginDiv from './components/loginVue/PageLoginDiv'
import PageArticle from './components/PageArticle'
import PagePicture from './components/PagePicture'

export default {
    data () {
        this.getRequestParam();
        this.initAction();
        return {
            loginShow : false,
            appShow : false,
            pictureShow : false,
            loadingShow : true,
            requestParam :{},
            personInfo :{}
        }
    },
    components: {
        PageLoginDiv,PageArticle,PagePicture
    },
    //这里，即父类可以调用到子类的方法对象：methods,进而调用methods中的成员方法
    methods: {
        //显示目录树
        switchLoginPage: function(){
            this.loginShow = !this.loginShow;
            this.appShow = true;
            //请求远程目录数据
            this.$children[1].toInitListDiv(this.personInfo);
        },
        showPictureSlide: function(_currentPic,_picArray){
            this.appShow = false;
            this.pictureShow = true;
            this.$children[2].showPictureArray(_currentPic,_picArray);
        },
        closePictureArray: function(){
            this.pictureShow = !this.pictureShow;
            this.appShow = true;
            this.$children[1].dealArticleVoice();
        },
        test:function(){
            console.log("test");
        },
        //组织出请求连接中的参数
        getRequestParam : function () {
            var args = new Object();
            var query = location.search.substring(1); // Get query string
            var pairs = query.split("&"); // Break at ampersand
            for (var i = 0; i < pairs.length; i++) {
                var pos = pairs[i].indexOf('='); // Look for "name=value"
                if (pos == -1) continue; // If not found, skip
                var argName = pairs[i].substring(0, pos); // Extract the name
                var value = pairs[i].substring(pos + 1); // Extract the value
                value = decodeURIComponent(value); // Decode it, if needed
                args[argName] = value; // Store as a property
            }
            this.requestParam = args;
        },
        //用id发起远程请求
        initAction : function(){
            let id = this.requestParam.id;
            this.requestLoginInfo(id);
        },
        requestLoginInfo : function(_data){
             var _self = this;
             $.ajax({
                 type : "POST",
                 url : "http://localhost:8081/shareWeb/doc/loginInfo.json",
                 data : _data,
                 dataType : "jsonp",
                 jsonp:"callback",
                 async : false,
                 jsonpCallback:"jsonpCallback",
                 success : function(data) {
                     _self.jsonpCallBack(data);
                 },
                 error:function(data){
                      console.log(data);
                 }
             })
        },
        //远程请求服务器之后的回调函数
        jsonpCallBack : function(_data){
            var _self = this;
            _self.loadingShow = false;
            console.log(this);
            console.log("_data.type:" + _data.type);
            if(_data.type=="1"){
                _self.loginShow = true;
                _self.personInfo = {name:"Atta的分享",id:"14101508"};
            }else{
                _self.appShow = true;
                _self.personInfo = {name:"Atta的分享",id:"14101508"};
                _self.$children[1].switchToArticlePage(_data.content);
            }
        }
    }
}
</script>
<style>
html {
  width:100%;
  height: 100%;
  margin:0 auto;
}
body {
  width:100%;
  height: 100%;
  margin:0 auto;
  font: normal 100% Helvetica, Arial, sans-serif;
}
*{
    margin:0;
    padding:0;
}
@media screen and (min-width:480px) {
    html,body{
        font-size:18px;
    }
}
@media screen and (min-width:640px) {
    html,body{
        font-size:24px;
    }
}
@media screen and (min-width:960px) {
    html,body{
        font-size:36px;
    }
}
.login{
    position: absolute;
    z-index: 101;
    width: 100%;
    height: 100%;
    opacity: 1;
}
#app {
    position: absolute;
    margin:0 auto;
    height : 100%;
    width:100%;
    z-index: 101;
}
#allPic{
    position: absolute;
    z-index: 101;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    opacity: 1;
}
#loading{
    position: absolute;
    z-index: 101;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    opacity: 1;
    text-align:center;
}
.loadingContent{
    position: absolute;
    z-index: 101;
    top: 45%;
    left:25%;
    width: 50%;
    height: 1rem;
    font-size:2rem;
}
.borderLing {
    border-bottom: 1px solid gray;
}
</style>
