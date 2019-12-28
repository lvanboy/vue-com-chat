<template>
 <div>
     <h3>C组件：我的父亲是B组件</h3>
     <p>准备获取父元素的值：{{tmpParentVal}}</p>
     <button @click="onClick">点击按钮，子元素主动向父元素传值供访问和修改显示</button>
     <button @click="onClick1">点击按钮,子元素主动获取父元素值供显示而不能修改</button>
 </div>
</template>
<script>
export default {
    name:"C",
    data(){
        return {
            emitValue:`${this.$options.name}组件（子）向B组件（父）传值`,
            tmpParentVal:''
        }
    },
    methods:{
        onClick(){
            this.$emit('onclick',this.emitValue)
        },
        onClick1(){
            this.tmpParentVal = this.$parent.parentValue;
        }
    }
}
</script>
<style scoped>

</style>
