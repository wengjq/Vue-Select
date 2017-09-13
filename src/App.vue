<template>
  <div id="app">
    <div class="demo1">demo1(禁用)：</div>
    <select-component disabled placement="bottom" v-model="model1" style="width: 150px; margin-right: 30px;">
      <select-item-component v-for="item in cityList" :value="item.value" :key="item.value">{{ item.label }}</select-item-component>
    </select-component>

    <div class="demo2">demo2(正常选择)：</div>
    <select-component placement="bottom" v-model="model1" style="width: 150px; margin-right: 30px;">
      <select-item-component v-for="item in cityList" :value="item.value" :key="item.value">{{ item.label }}</select-item-component>
    </select-component>

    <div class="demo2">demo2(可搜索)：</div>
    <select-component filterable placement="bottom" v-model="model1" style="width: 150px">
      <select-item-component v-for="item in cityList" :value="item.value" :key="item.value">{{ item.label }}</select-item-component>
    </select-component>

  </div>
</template>

<script>
import SelectComponent from './components/Select';
import SelectItemComponent from './components/Select-Item';

export default {
  name: 'app',
  components: {
    SelectComponent,
    SelectItemComponent
  },
  data () {
    return {
      cityList: [
          {
              value: 'beijing',
              label: '北京市'
          },
          {
              value: 'shanghai',
              label: '上海市'
          },
          {
              value: 'shenzhen',
              label: '深圳市'
          },
          {
              value: 'hangzhou',
              label: '杭州市'
          },
          {
              value: 'nanjing',
              label: '南京市'
          },
          {
              value: 'chongqing',
              label: '重庆市'
          }
      ],
      model1: ''
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #333;
  font-size: 13px;
  margin: 60px;
}
.demo1, .demo2 {
  display: inline-block;
  vertical-align: middle;
}
</style>
