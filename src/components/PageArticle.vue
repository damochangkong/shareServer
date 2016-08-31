<template>
    <cont-head-div></cont-head-div>
    <!-- 这里是列表、长文、语音的切换的地方 -->
    <navigate-list v-show="showList"></navigate-list>
    <cont-list-div v-show="showList"></cont-list-div>

    <navigate-article v-show="showArticle"></navigate-article>
    <cont-article-div v-show="showArticle"></cont-article-div>

    <navigate-voice v-show="showVoice"></navigate-voice>
    <cont-voice-div v-show="showVoice"></cont-voice-div>
</template>
<script>
import ContHeadDiv from './ContHeadDiv'
import ContListDiv from './ContListDiv'
import ContArticleDiv from './ContArticleDiv'
import ContVoiceDiv from './ContVoiceDiv'
import NavigateList from './NavigateList'
import NavigateArticle from './NavigateArticle'
import NavigateVoice from './NavigateVoice'
import $ from 'jquery'
export default {
    data () {
        return {
            showList: true,
            showArticle: false,
            showVoice: false,
            mainItems:{}
        }
    },
    components: {
        ContHeadDiv,
        NavigateList,ContListDiv,//1,2
        NavigateArticle,ContArticleDiv,//3,4
        NavigateVoice,ContVoiceDiv //5,6
    },
    //这里，即父类可以调用到子类的方法对象：methods,进而调用methods中的成员方法
    methods: {
        showTargetContent : function(target){
            if(target=="list"){
                this.showList = true;
                //调用子类初始化方法
            }else{
                this.showArticle = true;
                //调用子类初始化方法
            }
        },
        //第一次进来默认初始化列表
        toInitListDiv : function(_personInfo){
            this.$children[2].initNoteList(_personInfo);
        },
        //显示长文
        switchToArticlePage: function(_content){
            this.showList = false;
            this.showVoice = false;
            this.showArticle = true;
            this.$children[4].initArticleData(_content);
        },
        showVoicePage: function(item){
            this.showArticle = false;
            this.showVoice = true;
            this.$children[6].getSong(item);
        },
        playVoiceSilence : function(item){
            this.$children[6].getSong(item);
        },
        backToArticle : function(){
            this.showArticle = true;
            this.showVoice = false;
            this.showList = false;
        },
        closeArticle : function(){
            this.showArticle = false;
            this.showVoice = false;
            this.showList = true;
        },
        //显示列表
        backToNoteListFromArticle : function(){
            this.showArticle = false;
            this.showList = true;
        },
        dealArticleVoice : function(){
            this.$children[4].dealArticleVoice();
        }
    }
}
</script>
