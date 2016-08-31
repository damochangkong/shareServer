<template>
    <div class="headDiv a">
        <div class="element logo"></div>
        <div class="element title"><span>{{headName}}</span></div>
        <div class="element download"></div>
    </div>
</template>
<style scoped>
.headDiv {
  position: fixed;
  z-index: 1000;
  background:#ffffff;
  width: 100%;
  height: 7%;
  top:0;
  border-bottom: 1px solid gray;
}
.element{
  z-index: 101;
  width:2rem;
  height:2rem;
  background-size:100% 100%;
  top :0;
  bottom:0;
  margin:2.3%  5%;
}
.logo {
  float:left;
  background-image:url(./images/logo.png);
}
.title {
  float:left;
  width:6em;
}
.title span{
  line-height :2rem;
}
.download{
  float:right;
  background-image:url(./images/download.png);
}
</style>
<script>
module.exports = {
    data () {
      return {
        headName: '蜘蛛笔记'
      }
    }
}
</script>
