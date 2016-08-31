<template>
    <div class="navigateArticle addss">
        <div class="navigation borderLing " v-if="showOneNavigate">
            <ul class="borderLing">
                <li v-for="item in navigates">
                    <span >{{item.name}}</span>
                    <span class="splitTip" v-if="$index!=navigates.length-1">></span>
                </li>
            </ul>
        </div>
        <div class="navigationWord borderLing" v-if="!showOneNavigate" v-on:click="closeArticle()">
            <span><img src="./images/articleClose.png"></img></span>
            <span>关闭</span>
        </div>
    </div>
</template>
<script>
import $ from 'jquery'
export default {
    data () {
        return {
            showOneNavigate : true,
            navigates : []
        }
    },
    methods: {
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
        closeArticle : function(){
            this.$parent.closeArticle();
        }
    }
}
</script>
<style scoped>
*{
    margin:0;
    padding:0;
}
.navigateArticle{
    position: fixed;
    width:95%;
    height: 7%;
    background:#ffffff;
    padding-left:5%;
    z-index: 1000;
    top:7%;
}

.navigation{
    width:100%;
    height:100%;
}
.navigation ul{
    list-style:none;
    width:100%;
    height:100%;
}
.navigation ul li{
    text-align:left;
    float:left;
    height:100%;
    line-height:3.4rem;
}
.navigation ul li .splitTip{
    display:inline;
    margin:0 0.2rem;
    padding:0;
}

.navigationWord{
  display:block;
  width:100%;
  height:100%;
  line-height:3.4rem;
  background:#ffffff;
  z-index: 301;
}
.navigationWord span img{
  width:0.8rem;
  height:0.8rem;
}
.navigationWord span{
  float:left;
  color:blue;
  font-size:1rem;
  margin-right:0.5rem;
}
</style>
