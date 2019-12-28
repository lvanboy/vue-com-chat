<template>
  <div id="app" style="border:1px solid red;padding:10px;">
    <p>我是root，我引用了A和E组件</p>
    <div style="display:inline-block;">
      <A :msg="msg" ref="A" @toAlert="Alert" />
      <button @click="onClick" style="margin:10px;">点击后，修改A组件data中refValue值</button>
    </div>
    <div style="display:inline-block;margin-left:10px;">
      <E :msg1="msg1" />
      <button @click="onClick1" style="margin:10px;">点击后，修改E组件data中childValue值</button>
    </div>
    <div>
      <button @click="onClick2">点击按钮，App组件向D组件（子孙或后代）provide传值</button>
    </div>
    <p>来自D组件（子孙）向App传的值：</p>
  </div>
</template>

<script>
import A from "./components/A";
import E from "./components/E";
import Vue from "vue";
export default {
  name: "app",
  data() {
    return {
      val: "默认值",
      val1: "",
      val2: "默认值",
      msg: `子组件props方式：${this.$options.name}（父）向A组件（子）传递属性msg`,
      msg1: `子组件$attrs方式：${this.$options.name}（父）向E组件（E）传递属性msg1`
    };
  },
  components: {
    A,
    E
  },

  provide() {
    this.val = Vue.observable({ Val: "provide中注册一个响应值" });
    this.val1 = Object.defineProperty({}, "Val", {
      value: "初始值",
      writable: true,
      configurable: true,
      enumerable: false
    });
    this.data = new Proxy(this.$data, {
      get: (target, name) => {
        return name in target ? target[name] : "undefined";
      }
    });
    return {
      val: this.val,
      val1: this.val1,
      data: this.data
    };
  },

  methods: {
    Alert() {
      alert("来自APP组件的弹框");
    },
    onClick() {
      this.$refs["A"].refValue = `${this.$options.name}修改了A组件的refValue值`;
    },
    onClick1() {
      for (let i in this.$children) {
        if (this.$children[i].$options.name === "E") {
          this.$children[
            i
          ].childValue = `${this.$options.name}修改了E组件的childValue`;
        }
      }
    },
    onClick2() {
      alert("修改provide中的val值");
      this.val.Val = "默认值=>修改provide中的val值";
      this.val1.Val = "修改provide的val1值";
      this.data.val2 = "修改provide的val2值";
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
