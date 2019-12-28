<template>
 <div style="border:1px solid #787878;">
     <h2>F组件：C、D组件与我的关系难以描述</h2>
     <button @click="onClick" style="margin:10px;">点击按钮，向D组件传值</button>
 </div>
</template>
<script>
export default {
    name:"F",
    data(){
        return {
            mi:'F组件data中的值'
        }
    },
    methods:{
        onClick(){
            this.$root.$emit('ftod',this.mi)
        }
    }
}
</script>
<style scoped>
 
</style>
