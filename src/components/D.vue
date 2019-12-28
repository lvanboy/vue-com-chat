<template>
 <div>
     <h3>D组件：我的父亲是B组件</h3>
     <p >准备接受根组件inject值:{{val.Val}}</p>
     <p>通过defineproperty，inject传递的值：{{val1.Val}}</p>
     <p>通过proxy,inject传递的值：{{data.val2}}</p>
     <p>准备接受F组件（复杂关系）的值：{{localVal}}</p>
     <button @click="$listeners.toAlert">D组件：调用app组件（根）的方法</button>
 </div>
</template>
<script>
export default{
    name:"D",
    inject:['val','val1','data'],
    data(){
        return {
            localVal:""
        }
    },
    mounted() {
        this.$root.$on('ftod',(e)=>{
            this.localVal = e;
        })
    },

}
</script>
<style scoped>
 
</style>
