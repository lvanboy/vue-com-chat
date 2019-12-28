<template>
 <div style="border:1px solid #787878;padding:10px;">
     <h1>E组件：A组件是我兄弟,F组件是我儿子</h1>
     <p>{{$attrs.msg1}}</p>
     <p>{{childValue}}</p>
     <p>准备接受兄弟组件A的值：{{brotherVal}}</p>
     <button @click="onClick">点击按钮，E组件向A组件传值</button>
     <F/>
 </div>
</template>

<script>
import F from './F'
export default {
    name:"E",
    components:{
        F
    },
    data(){
        return{
            childValue:`${this.$options.name}组件中data的childValue值`,
            brotherVal:'',
            
        }
    },
    methods:{
        onClick(){
            this.$root.$emit('eclick',this.childValue);
        }
    },
    mounted(){
        this.$parent.$on('aclick',(e)=>{
            this.brotherVal = e;
            console.log(e)
        })
    }
}
</script>
<style scoped>
 
</style>
