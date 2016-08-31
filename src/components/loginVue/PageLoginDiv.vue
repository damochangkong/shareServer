<template>
    <cont-head-div></cont-head-div>
    <div class="content dds">
        <div class="tipShare tipShare1">
            <div class=" tipShare1img"></div>
            <span>{{authorName}}分享了他的蜘蛛笔记</span>
        </div>
        <div class="tipShare tipShare2"><span>请输入密码:</span></div>
        <div class="tipShare tipShare3">
            <input type="password" id="password" value="" v-model="password"></input>
            <div class="tipShare3Button" v-on:click="loginIn()"><span>确定</span></div>
        </div>
    </div>
</template>
<script>
import ContHeadDiv from '../ContHeadDiv.vue'
import $ from 'jquery'
export default {
    data () {
      return {
        authorName: "小芳",
        password: ""
      }
    },
    components: {
      ContHeadDiv
    },
    //这里，即父类可以调用到子类的方法对象：methods,进而调用methods中的成员方法
    methods: {
        loginIn: function(){
             var _self = this;
             $.ajax({
                 type : "POST",
                 url : "http://localhost:8081/shareWeb/doc/treeLong.json",
                 data : {},
                 dataType : "jsonp",
                 jsonp:"callback",
                 async : false,
                 jsonpCallback:"jsonpCallback",
                 success : function(data) {
                     _self.jsonpCallback(_self,data);
                 },
                 error:function(data){
                      console.log(data);
                 }
             })
        },
        //请求了远程数据的回调函数，负责生产主目录等
        jsonpCallback: function(_self,data){
            //登陆成功
            if(data.flag = "success"){
                console.log("success");
                this.$parent.switchLoginPage();
            }
        },
        invokeFatherToShowPicture: function(){
             this.$parent.$parent.showPictureSlide();
        }
    }
}
</script>
<style scoped>
*{
    margin:0;
    padding:0;
}
img{
    width:100%;
    height:100%;
}
.input{
    margin:0;
    padding:0;
    border:0;
}
.content {
    position: absolute;
    z-index: 100;
    width: 100%;
    height: 93%;
	  margin:0 auto;
    background-color:#ffffff;
    top: 7%;
}
.tipShare{
    width:95%;
    margin:5%;
    float:left;
    height: 3rem;
    line-height:3rem;
    font-size:1.5rem;
}
.tipShare1img{
    float:left;
    width: 3rem;
    height: 3rem;
    background-image:url(../images/lufei.jpg);
    background-size:100% 100%;
    border-radius:0.5rem;
}
.tipShare1 span{
    float:left;
    margin-left:1rem;
}
.tipShare3 input{
    display:block;
    float:left;
    width:25%;
    height:90%;
    font-size:2rem;
    overflow:hidden;
}
.tipShare3Button{
    width: 30%;
    float:left;
    background-color:#18bd83;
    border-radius:0.5rem;
    text-align:center;
    height: 3rem;
    line-height: 3rem;
    font-size:2rem;
    margin:0 10%;
}
</style>
