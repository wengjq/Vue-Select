<template>
  <div id="app">
    <div class="line">
      <div class="demo1">demo1(禁用)：</div>
      <select-component disabled placement="bottom" v-model="model1" style="width: 150px; margin-right: 30px;">
        <select-item-component v-for="item in cityList" :value="item.value" :key="item.value">{{ item.label }}</select-item-component>
      </select-component>

      <div class="demo2">demo2(正常选择)：</div>
      <select-component placement="bottom" v-model="model1" style="width: 150px; margin-right: 30px;">
        <select-item-component v-for="item in cityList" :value="item.value" :key="item.value">{{ item.label }}</select-item-component>
      </select-component>

      <div class="demo3">demo3(可搜索)：</div>
      <select-component filterable placement="bottom" v-model="model1" style="width: 150px; margin-right: 30px;">
        <select-item-component v-for="item in cityList" :value="item.value" :key="item.value">{{ item.label }}</select-item-component>
      </select-component>
    </div>  
    <div class="line">
      <div class="demo4">demo4(项的禁用)：</div>
      <select-component placement="bottom" v-model="model1" style="width: 150px; margin-right: 30px;">
        <select-item-component value="beijing" label="北京市"></select-item-component>
        <select-item-component value="shanghai" disabled>上海市</select-item-component>
        <select-item-component value="shenzhen">深圳市</select-item-component>
      </select-component>

      <div class="demo5">demo5(二级选择，可搜索)：</div>
      <select-component filterable placement="bottom" v-model="model2" style="width: 150px; margin-right: 30px;">
        <select-item-component value="beijing" label="北京市"></select-item-component>
        <dropdown-component second-select trigger="hover" placement="right-start">
            <select-item-component show-icon second-select value="guangzhou">广州</select-item-component>
            <second-select-menu-component slot="list">
                <select-item-component value="haizhu">海珠区</select-item-component>
                <select-item-component value="tianhe">天河区</select-item-component>
            </second-select-menu-component>    
        </dropdown-component>
        <select-item-component value="shanghai">上海市</select-item-component>
        <select-item-component value="shenzhen">深圳市</select-item-component>
      </select-component>

      <div class="demo6">demo6(多级选择，可搜索)：</div>
      <select-component filterable placement="bottom" v-model="model3" style="width: 150px; margin-right: 30px;">
        <select-item-component value="beijing" label="北京市"></select-item-component>
        <dropdown-component second-select trigger="hover" placement="right-start">
            <select-item-component show-icon second-select value="guangzhou">广州</select-item-component>
            <second-select-menu-component slot="list">
                <select-item-component value="baiyun">白云区</select-item-component>
                <select-item-component value="tianhe">天河区</select-item-component>
                <dropdown-component second-select trigger="hover" placement="right-start">
                  <select-item-component show-icon second-select value="haizhu">海珠区</select-item-component>
                  <second-select-menu-component slot="list">
                      <select-item-component value="pazhou">琶洲</select-item-component>
                      <select-item-component value="chigang">赤岗</select-item-component>
                  </second-select-menu-component>    
              </dropdown-component>
            </second-select-menu-component>    
        </dropdown-component>
        <select-item-component value="shanghai">上海市</select-item-component>
        <select-item-component value="shenzhen">深圳市</select-item-component>
      </select-component>
    </div>
  </div>
</template>

<script>
import SelectComponent from './components/Select';
import SelectItemComponent from './components/Select-Item';
import DropdownComponent from './components/Dropdown';
import SecondSelectMenuComponent from './components/Second-Select-Menu';

export default {
  name: 'app',
  components: {
    SelectComponent,
    SelectItemComponent,
    DropdownComponent,
    SecondSelectMenuComponent
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
      model1: '',
      model2: '',
      model3: ''
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
.demo1, .demo2, .demo3, .demo4, .demo5, .demo6 {
  display: inline-block;
  vertical-align: middle;
}
.line {
  margin-bottom: 10%;
}
</style>
