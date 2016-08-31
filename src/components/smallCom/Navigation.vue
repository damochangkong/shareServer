<template>
    <div class="navigation borderLing d">
        <ul class="borderLing">
            <li v-for="item in navigates">
                <span>{{item.name}}</span>
            </li>
        </ul>
    </div>
</template>
<script>
    export default {
        data () {
            return {
                navigates:[]
            }
        }
    }
</script>
<style scoped>
* {
  margin:0;
  padding:0;
}
.navigation{
    width:95%;
    height:10%;
    background:#ffffff;
    float:left;
    padding-left:5%;
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
    line-height:4.2rem;
}
.navigation ul li span{
    display:inline;
    margin:0;
    padding:0;
}
</style>
