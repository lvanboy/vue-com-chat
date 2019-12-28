<template>
  <div style="border:1px solid #ccc;padding:10px;">
    <h2>B组件：我引用了C和D组件</h2>
    <div style="display:inline-block;border:1px dashed #aaa;">
      <C @onclick="onClick($event)" />
      <p>准备接收子元素的值：{{childValue}} </p>
    </div>
    <div style="display:inline-block;border:1px dashed #aaa;">
      <D v-on="$listeners"/>
    </div>
  </div>
</template>

<script>
import C from "./C";
import D from "./D";
export default {
  name: "B",
  components: {
    C,
    D
  },
  data() {
    return {
      childValue: "",
      parentValue:"parentValue"
    };
  },
  methods: {
    onClick(e) {
        this.childValue =e;
    }
  },
  mounted(){
  }
};
</script>
<style scoped>
</style>
