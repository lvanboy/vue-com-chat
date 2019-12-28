<template>
 <div style="border:1px solid #343443;padding:20px;">
     <h1>A组件：B组件是我儿子</h1>
     <p>{{msg}}</p>
     <p>{{refValue}}</p>
     <p>准备接受E组件（兄）的值：{{brotherVal}}</p>
     <button @click="onClick">点击按钮，A组件向兄弟组件E组件传值</button>
     <B v-on="$listeners"/>
     <button style="margin:10px;" @click="$listeners.toAlert">A组件：调用app组件（根）的方法</button>
 </div>
</template>

<script>
import B from "./B";
export default {
    name:"A",
    props:{
        msg:String
    },
    data(){
        return {
            refValue:`${this.$options.name}组件data中refValue值`,
            brotherVal:''
        }
    },
    methods:{
        onClick(){
            this.$parent.$emit('aclick',this.refValue);
        }
    },
    components:{
        B
    },
    mounted(){
        this.$root.$on('eclick',(e)=>{
            this.brotherVal = e;
        })
    },
}
</script>
<style scoped>
 
</style>
