<template>
  <div class="date-picker" v-click-outside>
    <div class="picker-input">
      <span class="input-prefix">
        <i class="iconfont icon-date"></i>
      </span>
      <input type="text" :value="chooseDate" />
    </div>
    <div class="picker-panel" v-if="showPanel">
      <div class="picker-arrow" />
      <div class="picker-body">
        <div class="picker-header">
          <span class="picker-btn iconfont icon-prev-year" @click="onChangeYear('prev')"/>
          <span class="picker-btn iconfont icon-prev-month"  @click="onChangeMonth('prev')"/>
          <span class="picker-date">{{ showDate.year }}年{{ showDate.month+1 }}月</span>
          <span class="picker-btn iconfont icon-next-month"
          @click="onChangeMonth('next')"/>
          <span class="picker-btn iconfont icon-next-year" @click="onChangeYear('prev')"/>
        </div>
        <div class="picker-content">
          <div class="picker-weeks">
            <div v-for="week in ['日', '一', '二', '三', '四', '五', '六']" :key="week">{{ week }}</div>
          </div>
          <div class="picker-days">
            <div
              v-for="date in showDay"
              :key="date.getTime()"
              :class="{'other-month':!isCur(date).month,
              'is-select':isCur(date).select,
              'is-today':isCur(date).today}"
              @click="onchooseClick(date)"
            >{{ date.getDate() }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  // 自定义指令
  directives: {
    "click-outside": {
      // 只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
      // el(指令所绑定的元素，可以用来直接操作DOM)---->为钩子函数的参数
      // vnode---(Vue 编译生成的虚拟节点)
      bind(el, binding, vnode) {
        const vm = vnode.context;

        document.onclick = function(e) {
          const dom = e.target;
          // console.log(dom ,el);

          //contains el里面是否包含了dom 返回值Boolean值
          const isSon = el.contains(dom);

          if (isSon && !vm.showPanel) {
            vm.changPanel(true);
          } else if (!isSon && vm.showPanel) {
            vm.changPanel(false);
          }
        };
      }
    }
  },
  //model选项
  model:{
    // 绑定选项特性的名字
    prop:'date',
    //监听事件的名字
    event:'choose-dates',

  },
  props: {
    date: {
      type: Date,
      // 默认值
      default: () => new Date()
    }
  },
  data() {
    return {
      showPanel: false,
      showDate: {
        year: 0,
        month: 0,
        day: 0
      }
    };
  },
  methods: {
    changPanel(flag) {
      this.showPanel = flag;
    },
    //初始化showDate的年月日
    getShowDate(date) {
      const { year, month, day } = this.getYearMonthDay(date);

      this.showDate = {
        year,
        month,
        day
      };
    },
    // 封装函数
    getYearMonthDay(date) {
      const year = date.getFullYear();
      const month = date.getMonth();
      const day = date.getDate();
      return {
        year,
        month,
        day
      };
    },
    //上个月下个月天数颜色变浅
    isCur(date) {
      const chooseDate=new Date(this.chooseDate);
      const{year:chooseYear,month:chooseMonth,day:chooseDay}=this.getYearMonthDay(chooseDate);
      // 当天的年 月 日
      const{year:curYear,month:curMonth,day:curDay}=this.getYearMonthDay(new Date());
      // 这个月的年 月
      const { year: showYear, month: showMonth } = this.showDate;
      //for循环里面的年月日
      const { year, month,day } = this.getYearMonthDay(date);
      return {
        month: year == showYear && month == showMonth,
        select:year == chooseYear && month == chooseMonth && day==chooseDay,
        today:year == curYear && month == curMonth && day==curDay,
      };
    },
    // 点击日期
    onchooseClick(date){
      console.log("chooseClick----"+date)
      this.$emit("choose-dates",date); //自定义事件 子组件向父组件传递数据
      this.changPanel(false);//关闭日期
      this.getShowDate(date);
    },
    // 点击单箭头---切换月份
    onChangeMonth(type){
      const moveMonth = type ==='prev' ? -1 : 1;
      const {year,month,day}=this.showDate;
      const showDates=new Date(year,month,day);
      // 利用newDate()里面的方法setMoth修改月份
      showDates.setMonth(month+moveMonth);
      const {year:showYear,month:showMonth}=this.getYearMonthDay(showDates);
      this.showDate.year=showYear;
      this.showDate.month=showMonth;
    },
    // 点击双箭头 ---切换年份
    onChangeYear(type){
      const moveyear=type=='prev'?-1:1;
      this.showDate.year+=moveyear;
    }
  },
  computed: {
    //input框 默认显示今天
    chooseDate() {
      const { year, month, day } = this.getYearMonthDay(this.date);

      return `${year}-${month + 1}-${day}`;
    },
    //显示天数（上个月，这个月，下个月）
    showDay() {
      const { year, month } = this.showDate;
      const firstDay = new Date(year, month, 1);
      const week = firstDay.getDay(); //第一天是周几
      //上个月几号
      const startDay = firstDay - week * 24 * 60 * 60 * 1000;
      var arr = [];

      // 一天的毫秒数 ---> 24*60*60*1000
      for (let i = 0; i < 42; i++) {
        //  从上个月几号开始遍历 直到42个空位完
        arr.push(new Date(startDay + i * 24 * 60 * 60 * 1000));
      }

      return arr;
    },
  },

  created() {
    this.getShowDate(this.date);
  }
};
</script>

<style scoped>
@import "./assets/font.css";

.date-picker {
  display: inline-block;
}

.picker-input {
  position: relative;
}

.picker-input input {
  height: 40px;
  line-height: 40px;
  padding: 0 30px;
  background-color: #fff;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  outline: none;
  cursor: pointer;
}

.picker-input .input-prefix {
  position: absolute;
  left: 5px;
  width: 25px;
  height: 100%;
  line-height: 40px;
  text-align: center;
  color: #c0c4cc;
}

.picker-panel {
  position: absolute;
  width: 322px;
  height: 329px;
  margin-top: 5px;
  border: 1px solid #e4e7ed;
  border-radius: 4px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  background-color: #fff;
}

.picker-panel .picker-arrow {
  position: absolute;
  top: -12px;
  left: 30px;
  width: 0;
  height: 0;
  border: 6px solid transparent;
  border-bottom-color: #ebeef5;
}

.picker-panel .picker-arrow::after {
  position: absolute;
  left: -6px;
  top: 1px;
  content: "";
  display: block;
  width: 0;
  height: 0;
  border: 6px solid transparent;
  border-bottom-color: #fff;
  border-top-width: 0;
}

.picker-panel .picker-body {
}

.picker-panel .picker-header {
  display: flex;
  align-items: center;
  justify-content: center;
  padding-top: 15px;
  padding-bottom: 10px;
}
.picker-panel .picker-btn {
  margin-right: 5px;
  margin-left: 5px;
  font-size: 12px;
  color: #303133;
  cursor: pointer;
}

.picker-panel .picker-date {
  margin-left: 60px;
  margin-right: 60px;
  font-size: 14px;
  user-select: none;
}

.picker-panel .picker-content {
  padding: 0 10px 10px 10px;
  color: #606266;
  user-select: none;
}

.picker-panel .picker-weeks {
  display: flex;
  justify-content: space-around;
  height: 40px;
  line-height: 40px;
  border-bottom: 1px solid #ebeef5;
}

.picker-panel .picker-days {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
}

.picker-panel .picker-days div {
  width: 30px;
  height: 30px;
  line-height: 30px;
  text-align: center;
  margin: 4px 6px;
  font-size: 12px;
  cursor: pointer;
}

.picker-panel .picker-days div:hover {
  color: #409eff;
}

.picker-panel .picker-days div.is-today {
  color: #409eff;
  font-weight: 700;
}

.picker-panel .picker-days div.is-select {
  border-radius: 50%;
  background-color: #409eff;
  color: #fff;
}

.picker-panel .picker-days div.other-month {
  color: #c0c4cc;
}
</style>