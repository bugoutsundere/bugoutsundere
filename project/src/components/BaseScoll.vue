<template>
  <div class="vehicle-driven-mileage-details">
    <div
      ref="wrapper"
      class="list-wrappers"
      @mouseenter="mouseenterFunc"
      @mouseleave="mouseleaveFunc"
      @mousewheel="mousewheelFunc"
    >
      <div
        ref="loadList"
        :style="{ transform: getScrollDistance }"
        class="list-load"
      >
        <div
          v-for="(item, index) in dataList"
          ref="listItem"
          :key="`v${index}`"
          class="base-scroll-list-item"
          @click="scrollClick(item)"
        >
          <slot></slot>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    // 滚动数据列表
    dataList:{
      type:Array,
      default(){
        return [];
      }
    },
    // 滚动步数
    steep: {
      type:[String,Number],
      default:1
  },
  data() {
    return {
      /// --滚动参数
      isCanScroll: false, // 是否可以滚动
      scrollDistance: 0, // top距离
      isStop: false, // 是否停止
      wrapperHeight: 0, // 遮罩层高度
      loadList: 0,// 加载列表高度
      animationFrame: null,// 动画定时器标识
      singleWaitTimer: null,// 单步定时器标识
      waitTime: 5000,// 单步延时
      timer: null,
      diyStyleList: [],
    };
  },
  computed: {
    getScrollDistance() {
      return `translate(0px, ${this.scrollDistance}px)`;
    },
    isCanScroll() {
      if (this.viewableListHeight > this.wrapperHeight) {
        return true;
      } else {
        return false;
      }
    }
  },
  created() {
  },
  mounted() {
    // this.initScroll();
    // window && window.addEventListener('resize', this.initScroll);
  },
  beforeDestroy() {
    this.clear();
  },
  methods: {
    // 初始化滚动条
    initScroll() {
      this.$nextTick(() => {
        this.clearAnimation();
        this.wrapperHeight =
          this.$refs.wrapper && this.$refs.wrapper.clientHeight;
        this.loadListHeight =
          this.$refs.loadList && this.$refs.loadList.clientHeight;
        this.singleHeight = Number(this.getSingleHeight());
        this.startScroll();
      });
    },
    // 获取单步高度（包括高度+外边距）
    getSingleHeight() {
      let item = document.getElementsByClassName("base-scroll-list-item");
      if (!(item && item[0])) return 0;
      let Height = window.getComputedStyle(item[0]).height.slice(0, -2);
      let MarginBottom = window
        .getComputedStyle(item[0])
        .marginBottom.slice(0, -2);
      return (Number(Height) + Number(MarginBottom) + 2).toFixed(0);
    },
    startScroll() {
      if (!this.isCanScroll) return;
      this.isStop = false;
      this.run();
    },
    stopScroll() {
      this.isStop = true;
      this.singleWaitTimer = null;
      this.singleWaitTimer && clearTimeout(this.singleWaitTimer);
      this.clearAnimation();
    },
    run() {
      if (!this.isCanScroll) return;
      let that = this;
      that.clearAnimation();
      that.animationFrame = window.requestAnimationFrame(() => {
        let absScrollDistance = Math.abs(that.scrollDistance);
        if (that.scrollDistance > 0 || absScrollDistance > that.ListHeight) {
          that.scrollDistance = 0;
        }
        if (that.scrollDistance !== 0 || that.singleWaitTimer) {
          this.scrollDistance -= this.steep;
        }
        if (!that.isStop) {
          that._judgeSingle();
        }
      });
    },
    // 控制单步执行
    _judgeSingle() {
      this.singleWaitTimer && clearTimeout(this.singleWaitTimer);
      if (this.singleHeight) {
        // 延时执行
        console.log(this.scrollDistance, this.singleHeight);
        if (Math.abs(this.scrollDistance) % this.singleHeight === 0) {
          this.singleWaitTimer = setTimeout(() => {
            this.run();
          }, this.waitTime);
        } else {
          this.run();
        }
      }
    },
    isLoad() {},
    mouseenterFunc() {
      this.stopScroll();
    },
    mouseleaveFunc() {
      this.startScroll();
    },
    // 鼠标滚动事件
    mousewheelFunc(e) {
      if (!this.isCanScroll) return false;
      // 返回用户滚动方向，上为正，下为负(判断滚动方向不准确，如果要改为左右方向还需判断)
      let dis = e.deltaY;
      let rollerScrollDistance = 20;
      if (dis > 0) {
        this.scrollDistance -= rollerScrollDistance;
      }
      if (dis < 0) {
        this.scrollDistance += rollerScrollDistance;
      }
      console.log(this.scrollDistance);
      this.diyStyleList = [];
      this.run();
    },
    clearAnimation() {
      if (this.animationFrame) {
        cancelAnimationFrame(this.animationFrame);
        this.animationFrame = null;
      }
    },
    clear() {
      clearInterval(this.timer);
      this.timer = null;
    },
    // 动态计算样式
    setItemStyle() {
      // 可视区域第一行开始位置---确定每行只有3列
      // if (Math.abs(this.scrollDistance) % this.singleHeight !== 0) return;
      let firstStartIndex =
        (Math.abs(this.scrollDistance) / this.singleHeight) * 3;
      let secondStartIndex = firstStartIndex + 3;
      let thirdStartIndex = firstStartIndex + 6;
      let fourthStartIndex = firstStartIndex + 9;
      this.formatStyle(secondStartIndex, {});
      this.formatStyle(thirdStartIndex, { "third-style": true });
      this.formatStyle(fourthStartIndex, { "fourth-style": true });
      return firstStartIndex;
    },
    // 传入dom对象
    formatStyle(index, style) {
      this.$set(this.diyStyleList, [index], style);
      this.$set(this.diyStyleList, [index + 1], style);
      this.$set(this.diyStyleList, [index + 2], style);
    },
    // 点击进入详情页
    scrollClick(item) {
      this.startScroll();
      this.$router.push({
        path: "/LargeSrceenViewTrack",
        query: {
          car_id: item.car_id
        }
      });
      // this.$router.push({
      //   name: 'LargeSrceenViewTrack',
      //   params: {
      //     platenum: item.platenum
      //   }
      // });
    },
    debounce(func, wait) {
      let timeout;
      return function() {
        let context = this;
        let args = arguments;

        if (timeout) clearTimeout(timeout);

        timeout = setTimeout(() => {
          func.apply(context, args);
        }, wait);
      };
    },
    getDataList() {
      this.dataList = data;
      this.scrollDistance = 0;
      this.singleWaitTimer = null;
      this.initScroll();
      window && window.addEventListener("resize", this.initScroll);
    }
  }
};
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import "~@/styles/_large-screenView";
@import "~@/styles/convert-viewport";
* {
  box-sizing: border-box;
}
.vehicle-driven-mileage-details {
  background-color: black;
  overflow: hidden;
  width: 66vw;
  height: 100%;
  margin-left: vw(43);
  margin-right: vw(76);
  .n-left {
    width: vw(119);
    margin-left: vw(27);
  }
  .n-center {
    width: vw(143);
  }
  .n-right {
    width: vw(111);
  }
  .detail-description {
    display: flex;
    align-items: center;
    overflow: hidden;
    margin-top: vh(15);
    margin-bottom: vh(15);
    white-space: nowrap;
    text-overflow: ellipsis;
    -o-text-overflow: ellipsis;
  }
  .list-wrappers {
    overflow: hidden;
    height: vh(170);
    &::-webkit-scrollbar {
      display: none;
    }
  }
  .list-load {
    // position: relative;
    display: flex;
    flex-flow: wrap;
    justify-content: space-between;
    overflow: hidden;
    &::-webkit-scrollbar {
      display: none;
    }
    &:after {
      content: "";
      width: vw(400);
    }
    .item-wrap {
      width: 100%;
      // opacity: 0.5;
    }
    .item-wrap-flex {
      display: flex;
      align-items: center;
    }
    .base-scroll-list-item {
      display: flex;
      align-items: center;
      // overflow: hidden;
      width: vw(400);
      height: vh(40);
      background: rgba(0, 199, 251, 0.15);
      border: 1px solid rgba(0, 199, 251, 0.1);
      border-left: 1px solid #00c7fb;
      margin-bottom: vh(8);
      backdrop-filter: blur(10px);
      &::-webkit-scrollbar {
        display: none;
      }
      &:after {
        content: "";
        position: absolute;
        width: 0;
        height: 0;
        margin-left: vw(11);
        border-width: vh(6) 0 vh(6) vw(4);
        border-style: solid;
        border-color: transparent transparent transparent #00c7fb;
        // 高斯模糊
        filter: blur(0.2px);
      }
      &:hover {
        border: 1px solid #00c7fb;
        cursor: pointer;
      }
    }
    .third-style {
      opacity: 0.7;
    }
    .fourth-style {
      opacity: 0.5;
    }
    .vehicle-name {
      height: vh(14);
      font-family: PingFangSC-Regular;
      font-weight: Regular;
      font-size: vw(14);
      color: #ffffff;
      letter-spacing: 0;
      line-height: vh(14);
    }
    .vehicle-mileage {
      height: vh(18);
      font-family: DINAlternate-Bold;
      font-weight: Bold;
      font-size: vw(16);
      color: #ffffff;
      letter-spacing: 2px;
      line-height: vh(18);
    }
    .current-province {
      height: vh(14);
      font-family: PingFangSC-Regular;
      font-weight: Regular;
      font-size: vw(14);
      color: #00caff;
      letter-spacing: 0;
      line-height: vh(14);
    }
  }
}
</style>
