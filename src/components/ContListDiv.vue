<template>
    <div class="contentList ad" id="contentList">
        <ul>
            <template v-for="item in items">
                <li class="borderLing" v-on:click="showSubTree(item)">
                    <div class="logo logoCont" v-if="item.type=='content'"></div>
                    <div class="logo logoNode" v-if="item.type=='node'"></div>
                    <span class="title">{{ item.name }}</span>
                </li>
            </template>
        </ul>
    </div>
</template>

<script>
import $ from 'jquery'
export default {
     //这里定义本页的成员变量
     data(){
         return {
             msg: 'Hello World!!',
             mainListMap: new Map(), // 导航列表的map
             items:[],
             jsonMap: new Map(),
             personInfo : {}
         }
     },
     methods: {
         initNoteList : function(_personInfo){
             this.personInfo = _personInfo;
             this.personInfo.index = 0;
             this.mainListMap.set(_personInfo.id,_personInfo);
             this.requestRemoteData();
         },
         requestRemoteData: function(){
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
         jsonpCallback: function(_this,jsonObject){
              //显示主目录
              _this.items = jsonObject.items;
              //把各个目录放在jsonMap键值对里面
              if((typeof jsonObject.userId) == 'string'){
                  //设置目录层级
                  _this.addIndexIntoObject(_this,jsonObject,1);
                  //设置目录树，导航条
                  _this.searchJson(_this,jsonObject,1);
              }
         },
         //设置目录层级
         addIndexIntoObject: function(_self,jsonObject,index){
         		if(jsonObject instanceof Array){
                index = index+1;
                for(var i=0;i<jsonObject.length;i++){
                    _self.addIndexIntoObject(_self,jsonObject[i],index);
                }
         		}else{
                jsonObject['index'] = index;
                if((typeof jsonObject.items) == 'object'){
                    _self.addIndexIntoObject(_self,jsonObject.items,index);
                }
         		}
         },
         //遍历目录树，把节点目录放入jsnoMap中
         searchJson: function(_self,jsonObject) {
         		if(jsonObject instanceof Array){
                for(var i=0;i<jsonObject.length;i++){
                    _self.searchJson(_self,jsonObject[i]);
                }
         		}else{
                //目录节点放在jsonMap中
                if(jsonObject.type == "node" || (typeof jsonObject.userId) == 'string'){
                    //设置目录导航的map
                    if((typeof jsonObject.userId) == 'string'){
                        _self._createNewItemObject(_self.mainListMap,jsonObject);
                        _self._setNavigateList([..._self.mainListMap.values()]);
                        this.$nextTick(this.$parent.$children[1].addListener());
                    }
                    _self.jsonMap.set(jsonObject.id, jsonObject.items);
                    _self.searchJson(_self,jsonObject.items);
                }
         		}
         },
         //点击目录调用该方法，根据ID取jsonMap中的value
         showSubTree: function(item){
             var key = $.trim(item.id);
             var name = $.trim(item.name);
             var index= item.index;
             var items= $.trim(item.items);
             //this指向的是本页面的对象
             if(this.jsonMap.has(key)){
                 //显示子目录
                 this.items = this.jsonMap.get(key);
                 //设置目录树
                 this._createNavigate(this.mainListMap,key,name,index);
                 this._setNavigateList([...this.mainListMap.values()]);
             }else{
                 //显示长文页
                 this.$parent.switchToArticlePage(items);
             }
             this.$nextTick(this._setNavigateStyle);
         },
         _setNavigateList : function(items){
             this.$parent.$children[1].mainItems = items;
         },
         //点击导航
         navigate: function(key,index){
             if(index==0){
                return ;
             }
             key = $.trim(key);
             //this指向的是本页面的对象
             if((typeof this.jsonMap.get(key)) == 'object'){
                 //显示子目录
                 this.items = this.jsonMap.get(key);
                 //显示导航条
                 this.mainListMap = this._createNewMap(index,this.mainListMap);
                 this._setNavigateList([...this.mainListMap.values()]);
             }else{
                 //点击长文的导航的话，什么都不做？
             }
             this.$nextTick(this._setNavigateStyle);
         },
         _createNewItemObject: function(map,jsonObject){
             var itemObject = new Object();
             itemObject.name = jsonObject.name;
             itemObject.id = jsonObject.id;
             itemObject.index = jsonObject.index;
             map.set(itemObject.id,itemObject);
         },
         _createNavigate: function(map,id,name,index){
             var itemObject = new Object();
             itemObject.name = name;
             itemObject.id = id;
             itemObject.index = index;
             map.set(id,itemObject);
         },
         _createNewMap : function(_index,map){
             var newMap = new Map();
             newMap.set(this.personInfo.id,this.personInfo);
             for(let [key,value] of map){
                 if(value.index <= _index){
                     newMap.set(key,value);
                 }
             }
             return newMap;
         },
         //设置导航条的对齐方式
         _setNavigateStyle : function(){
             var parentWidth = $("#navigationList").width();
             var subWidth = this._calculateNavigate();
             var initWidth = $("#navigationList ul li").eq(0).width();
             var initLeft = "-" + 100*(initWidth/parentWidth) + "%";
             if(subWidth >= parentWidth){
                 var distance = "-" + 100*(subWidth-parentWidth)/parentWidth + "%";
                 $("#navigationList ul").css({left: distance});
             }else{
                 $("#navigationList ul").css({left: initLeft});
             }
         },
         _calculateNavigate : function(){
             var width = 0;
             $("#navigationList ul li").each(function(index){
                  width = $(this).width() + width;
             })
             return width;
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
.noteList{
  position: absolute;
  width:100%;
  height: 93%;
  top:7%;
  text-align:center;
  z-index:200;
}
li {
  float:left;
  display:block;
  height:4em;
  line-height:4em;
}


.contentList {
  position: absolute;
  width:100%;
  height:86%;
  top:14%;
  background:#ffffff;
  text-align:center;
  z-index:200;
  overflow:scroll;
}
.contentList ul {
  list-style:none;
  width:95%;
  height:100%;
  margin-left:5%;
}
.contentList ul li{
  text-align:left;
  width:100%;
}
.contentList ul li .logo{
  display:block;
  float:left;
  width:2rem;
  height:50%;
  margin:1rem auto;
}
.logoCont{
  background-image:url(./images/articleLogo.png);
  background-size:cover;
}
.logoNode{
  background-image:url(./images/noteLogo.png);
  background-size:cover;
}
.contentList ul li span{
  float:left;
  margin-left:1rem;
}
</style>
