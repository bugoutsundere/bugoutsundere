<template>
  <label>
    {{ value }}
    <input v-bind="$attrs" v-on="inputListeners" />
    <input
      type="checkbox"
      :checked="checked"
      @change="$emit('change', $event.target.checked)"
    />
    <input :value="value" @input="$emit('input', $event.target.value)" />
  </label>
  <!-- <input> -->
</template>

<script>
export default {
  // inheritAttrs: false,
  props: ["checked","value"],
  // model: {
  //   prop: "checked",
  //   event: "change"
  // },
  data() {
    return {
      label: ""
      // value: "",
    };
  },
  created() {
    console.log(this.$attrs);
  },
  computed: {
    inputListeners() {
      let vm = this;
      return Object.assign({}, this.$listeners, {
        input(event) {
          vm.$emit("input", event.target.value);
        }
      });
    }
  },
  watch: {
    checked(val, old) {
      console.log(val, old);
    }
  }
};
</script>

<style></style>
