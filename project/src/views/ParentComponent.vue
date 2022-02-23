<template>
  <div class="main">
    <div>ParentMain</div>
    <!-- 父组件模板传入prop值button和非prop值a，b -->
    <base-input v-model="loginVue" @focus="onFocus"></base-input>
    <children-component
      :data="data"
      :title="title"
      @update:title="title = $event"
    ></children-component>
  </div>
</template>

<script>
import ChildrenComponent from "./ChildrenComponent.vue";
export default {
  components: {
    ChildrenComponent
  },
  data() {
    return {
      data: [{ name: "test" }],
      loginVue: "",
      title: "sync"
    };
  },
  watch: {
    data(val, oldval) {
      // console.log("new", val, "old", oldval);
    }
  },
  methods: {
    onFocus(event) {
      console.log(event.target.value);
    }
  }
};
</script>

<style></style>
