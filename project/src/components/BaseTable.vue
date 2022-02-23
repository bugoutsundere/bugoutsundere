<template>
  <div @mouseenter="tableClick">BaseTable</div>
</template>

<script>
export default {
  data: function() {
    return {
      arrData: [1, 2, 3]
    };
  },
  created() {
    
  },
  methods:{
    tableClick(){
      this.arrData.push(4);
    }
  },
  updated: function() {
    console.log("1==我会先执行");
    this.$nextTick(function() {
      //在下次 DOM 更新循环结束之后执行这个回调。在修改数据之后立即使用这个方法，获取更新后的DOM.
      console.log("3==我只能等页面渲染完了才会立即执行");
    });
    console.log("2==我虽然在最后但会比$nextTick先执行");
  }
};
</script>
<style></style>
