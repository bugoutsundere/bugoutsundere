<template>
  <div>
    {{ title }}
    <button @click="changeMyData">Button</button>
    <BaseTable></BaseTable>
  </div>
</template>

<script>
export default {
  props: {
    data: Array,
    title: String,
    likes: Number,
    isPublished: Boolean,
    commentIds: Array,
    author: Object,
    callback: Function,
    contactsPromise: Promise, // or any other constructor
    propB: [String, Number], // 多个可能的类型
    // 必填的字符串
    // propC: {
    //   type: String,
    //   required: true
    // },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function() {
        return { message: "hello" };
      }
    },
    // 自定义验证函数
    propF: {
      validator: function(value) {
        // 这个值必须匹配下列字符串中的一个
        return ["success", "warning", "danger"].indexOf(value) !== -1;
      }
    }
  },
  data() {
    return {
      myData: JSON.parse(JSON.stringify(this.data))
    };
  },
  created() {
    // console.log(this.propF);
  },
  methods: {
    changeMyData() {
      this.myData.push(1);
      this.$emit("update:title", this.title + "1");
    }
  }
};
</script>

<style></style>
