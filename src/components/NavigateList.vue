<template>
    <div id="navigationList" class="navigationList borderLing ad">
        <ul class="borderLing">
            <li v-for="item in mainItems">
                <a v-on:click="invokeList(item.id,item.index)">{{ item.name }}</a>
                <span class="splitTip" v-if="$index!=mainItems.length-1">></span>
            </li>
        </ul>
    </div>
</template>
<script>
import $ from 'jquery'
export default {
    data () {
        return {
            authorName: "小芳",
            mainItems : [{name: 'XX的分享'}], // 导航列表
            startX : 0,
            startLeft : 0
        }
    },
    methods: {
        invokeList: function(key,index){
             this.$parent.$children[2].navigate(key,index);
        },
        addListener : function(){
            var target = document.getElementById("navigationList");
            target.addEventListener('touchstart', this._touchStart,false);
            target.addEventListener('touchmove', this._touchMove,false);
            target.addEventListener('touchend', this._touchEnd,false);
        },
        _touchStart : function(e){
            this.startLeft = parseInt($(this.$el).find("ul").css("left"));
            var touch = e.touches[0];
            this.startX = touch.pageX;//刚触摸时的坐标
        },
        //滑动
        _touchMove : function(e){
            var touch = e.touches[0];
            var distanceX = touch.pageX - this.startX;//滑动的距离
            var parentWidth = $(this.$el).width();
            var totalX = distanceX + this.startLeft;
            var nowLeft = 100*totalX/parentWidth + "%";
            $("#navigationList ul").css({left: nowLeft});
        },
        //手指离开屏幕
        _touchEnd : function(e){
            var nowLeft = parseInt($(this.$el).find("ul").css("left"));
            var parentWidth = $(this.$el).width();
            var initLeft = this._calcUlLeft(parentWidth);
            console.log("nowLeft:"+nowLeft + ",initLeft:" + initLeft);
            if(nowLeft > 0){
               $("#navigationList ul").css({left: "0%"});
            }
            if(initLeft < 0 && nowLeft < initLeft){
               $("#navigationList ul").css({left: initLeft});
            }
        },
        _calcUlLeft : function(_parentWidth){
            var subWidth = this._calculateWidth($("#navigationList ul li"));
            var distance = _parentWidth - subWidth;
            return distance;
        },
        _calculateWidth : function(array){
            var width = 0;
            array.each(function(index){
                 width = $(this).width() + width;
            });
            return width;
        }
    }
}
</script>
<style scoped>
li {
  float:left;
  display:block;
  height:4em;
  line-height:4em;
}
.navigationList{
  position: fixed;
  width:95%;
  height: 7%;
  background:#ffffff;
  margin-left:5%;
  z-index: 1000;
  top:7%;
  overflow:hidden;
}
.navigationList ul{
  position: absolute;
  list-style:none;
  width:300%;
  height:100%;
  left:0%;
}
.navigationList ul li{
  height:100%;
  text-align:left;
  line-height:3.4rem;
}
.navigationList .splitTip{
  display:inline;
  margin:0 0.3rem;
}
</style>
