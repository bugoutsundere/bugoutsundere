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
        ref="viewableList"
        :style="{ transform: getScrollDistance }"
        class="details-list"
      >
        <div
          v-for="(item, index) in dataList"
          ref="listItem"
          :key="`v${index}`"
          :class="diyStyleList[index]"
          class="details-list-item"
          @click="scrollClick(item)"
        >
          <div class="n-left vehicle-name">{{ item.car_id }}</div>
          <div class="n-center vehicle-mileage">{{ item.total_mileage }}km</div>
          <div class="n-right current-province">{{ item.city }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  // props: {
  // },
  data() {
    return {
      // 车辆行驶里程详情
      dataList: [],
      /// --滚动参数
      isCanScroll: false,// 是否可以滚动
      scrollDistance: 0, // top距离
      isStop: false, // 是否停止
      steep: 1, // 滚动步数
      wrapperHeight: 0,
      viewableListHeight: 0,
      animationFrame: null,
      // 单步定时器标识
      singleWaitTimer: null,
      // 单步延时
      waitTime: 5000,
      diyStyleList: [],
      cityInfo: {
        province: "",
        level: 1
      },
      timer: null
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
  watch: {
  },
  created() {
    this.getDataList();
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
        this.viewableListHeight =
          this.$refs.viewableList && this.$refs.viewableList.clientHeight;
        this.singleHeight = Number(this.getSingleHeight());
        this.startScroll();
      });
    },
    // 获取单步高度（包括高度+外边距）
    getSingleHeight() {
      let item = document.getElementsByClassName("details-list-item");
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
      const data = [
        {
          city: "广东省",
          total_mileage: 232548,
          platenum: "粤B19976",
          car_id: "100001"
        },
        {
          city: "北京市",
          total_mileage: 14816,
          platenum: "鄂A2295F",
          car_id: "53"
        },
        {
          city: "北京市",
          total_mileage: 1681,
          platenum: "粤BB5G13",
          car_id: "59"
        },
        {
          city: "北京市",
          total_mileage: 384,
          platenum: "鄂A2295G",
          car_id: "csds"
        },
        {
          city: "江苏省",
          total_mileage: 294,
          platenum: "蒙B67666",
          car_id: "1654_92"
        },
        {
          city: "上海市",
          total_mileage: 292,
          platenum: "蒙B65666",
          car_id: "1654_91"
        },
        {
          city: "上海市",
          total_mileage: 290,
          platenum: "蒙B64666",
          car_id: "1654_90"
        },
        {
          city: "浙江省",
          total_mileage: 288,
          platenum: "蒙B63666",
          car_id: "1654_89"
        },
        {
          city: "上海市",
          total_mileage: 286,
          platenum: "蒙B62666",
          car_id: "1654_88"
        },
        {
          city: "重庆市",
          total_mileage: 284,
          platenum: "蒙B61666",
          car_id: "1654_87"
        },
        {
          city: "上海市",
          total_mileage: 282,
          platenum: "蒙B96666",
          car_id: "1654_86"
        },
        {
          city: "重庆市",
          total_mileage: 280,
          platenum: "蒙B86666",
          car_id: "1654_85"
        },
        {
          city: "陕西省",
          total_mileage: 278,
          platenum: "蒙B76666",
          car_id: "1654_84"
        },
        {
          city: "上海市",
          total_mileage: 276,
          platenum: "蒙B56666",
          car_id: "1654_83"
        },
        {
          city: "上海市",
          total_mileage: 274,
          platenum: "蒙B46666",
          car_id: "1654_82"
        },
        {
          city: "陕西省",
          total_mileage: 272,
          platenum: "蒙B36666",
          car_id: "1654_81"
        },
        {
          city: "山西省",
          total_mileage: 270,
          platenum: "蒙B26666",
          car_id: "1654_80"
        },
        {
          city: "内蒙古",
          total_mileage: 268,
          platenum: "蒙B16666",
          car_id: "1654_79"
        },
        {
          city: "山西省",
          total_mileage: 266,
          platenum: "蒙B66665",
          car_id: "1654_78"
        },
        {
          city: "辽宁省",
          total_mileage: 264,
          platenum: "蒙B66664",
          car_id: "1654_77"
        },
        {
          city: "辽宁省",
          total_mileage: 262,
          platenum: "蒙B66663",
          car_id: "1654_76"
        },
        {
          city: "河南省",
          total_mileage: 260,
          platenum: "蒙B66662",
          car_id: "1654_75"
        },
        {
          city: "上海市",
          total_mileage: 258,
          platenum: "蒙B66661",
          car_id: "1654_74"
        },
        {
          city: "陕西省",
          total_mileage: 256,
          platenum: "蒙B66660",
          car_id: "1654_73"
        },
        {
          city: "山西省",
          total_mileage: 254,
          platenum: "蒙B66669",
          car_id: "1654_72"
        },
        {
          city: "内蒙古",
          total_mileage: 252,
          platenum: "蒙B66668",
          car_id: "1654_71"
        },
        {
          city: "吉林省",
          total_mileage: 250,
          platenum: "蒙B66667",
          car_id: "1654_70"
        },
        {
          city: "吉林省",
          total_mileage: 250,
          platenum: "蒙B66667",
          car_id: "1654_70"
        },
        {
          city: "吉林省",
          total_mileage: 250,
          platenum: "蒙B66667",
          car_id: "1654_70"
        },
        {
          city: "吉林省",
          total_mileage: 250,
          platenum: "蒙B66667",
          car_id: "1654_70"
        },
        {
          city: "吉林省",
          total_mileage: 250,
          platenum: "蒙B66667",
          car_id: "1654_70"
        },
        {
          city: "吉林省",
          total_mileage: 250,
          platenum: "蒙B66667",
          car_id: "1654_70"
        },
        {
          city: "吉林省",
          total_mileage: 250,
          platenum: "蒙B66667",
          car_id: "1654_70"
        },
        {
          city: "吉林省",
          total_mileage: 250,
          platenum: "蒙B66667",
          car_id: "1654_70"
        },
        {
          city: "吉林省",
          total_mileage: 250,
          platenum: "蒙B66667",
          car_id: "1654_70"
        },
        {
          city: "吉林省",
          total_mileage: 250,
          platenum: "蒙B66667",
          car_id: "1654_70"
        }
      ];
      // const { data } = await getUnder(this.cityInfo);
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
  .select-down {
    position: absolute;
    bottom: 0;
    left: 50%;
    width: vw(100);
    height: vw(100);
    // background: url("~@/assets/images/select_down.gif");
    background-size: 100% 100%;
    transform: translate(-10%, 15%);
  }
  // &:after {
  //   content: '';
  //   position: absolute;
  //   bottom: 0;
  //   left: 50%;
  //   width: vw(100);
  //   height: vw(100);
  //   background: url('~@/assets/images/select_down.gif');
  //   background-size: 100% 100%;
  //   transform: translate(-10%, 15%);
  // }
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
  .details-list {
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
    .details-list-item {
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
