<template>
    <div class="navigationLink borderLing">
        <span> < </span><a v-on:click="backToArticle()"><span>返回</span></a>
    </div>
</template>
<script>
import $ from 'jquery'
export default {
    data () {
        return {
            authorName: "小芳",
            mainItems : [{name: 'XX的分享'}], // 导航列表
        }
    },
    methods: {
        backToArticle : function(){
            //var audio = document.getElementById("audio");
            //audio.pause();
            this.$parent.backToArticle();
        }
    }
}
</script>
<style scoped>
*{
  margin:0;
  padding:0;
}
.navigationLink{
  position: fixed;
  width:95%;
  height: 7%;
  top:7%;
  line-height:3.5rem;
  text-align:left;
  background:#ffffff;
  padding-left:5%;
  z-index: 1000;
}
.navigationLink a{
  position:absolute;
  color:blue;
  margin-left:0.5rem;
}
</style>
