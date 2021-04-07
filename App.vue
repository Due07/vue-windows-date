<template>
  <div class="app">


    <!-- <date-picker :date="date" @choose-date="choose"/> 
      绑定特性 和监听事件  在组件里可以用 v-model  
      在子组件里写上选项  选项名为model:{ }
    -->
    <date-picker v-model="date"/>
  </div>
</template>

<script>
import DatePicker from './DatePicker';

export default {
  data(){
    return{
      date:new Date(),
    }
  },
  components: {
    DatePicker,
  },  
  created(){
    console.log(this.date);
  }
  // methods:{
  //   choose(date){
  //     this.date=date;
  //     console.log(this.date)
  //   }
  // }
}
</script>

<style scoped>

</style>