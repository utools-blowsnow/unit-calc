<template>
  <div class="UnitCalc">
    <container :title="currentUnit.name" >
      <template slot="head">
        <span style="margin: 0 10px;">待换算值：</span>
        <!-- 其实 `el-input-number` 也可以输入十六进制，只需以 `0x` 开头即可-->
        <el-input v-if="currentUnit.key == 'hex' && srcUnit == '十六进制'" size="mini" style="width: 120px;" v-model="value" placeholder="待换算值" @change="calc"></el-input>
        <el-input-number v-else size="mini" style="width: 120px;" v-model="value" placeholder="待换算值" @change="calc"></el-input-number>
        <el-select size="mini" style="width: 100px;margin-left: 20px;" v-model="srcUnit" placeholder="选择转换方式" @change="calc">
          <el-option
              v-for="item in unitTypes"
              :key="item"
              :label="item"
              :value="item">
          </el-option>
        </el-select>
        <span style="padding: 0 10px;">&lt;=&gt;</span>
        <el-select size="mini" style="width: 100px;" v-model="toUnit" placeholder="选择转换方式" @change="calc">
          <el-option label="全部" value="全部"></el-option>
          <el-option
              v-for="item in unitTypes"
              :key="item"
              :label="item"
              :value="item">
          </el-option>
        </el-select>
      </template>
      <el-row :gutter="20">
        <el-col :span="12" v-for="item in results">
          <div class="unit-result-item">
            <span style="float: left">{{ item.value }}</span>
            <span style="float: right">{{ item.unit }}</span>
            <span class="clearfix"></span>
          </div>
        </el-col>
      </el-row>

<!--      <el-table-->
<!--          :data="results"-->
<!--          style="width: 100%">-->
<!--        <el-table-column-->
<!--            prop="value"-->
<!--            label="数据"-->
<!--            min-width="180">-->
<!--        </el-table-column>-->
<!--        <el-table-column-->
<!--            prop="unit"-->
<!--            label="单位"-->
<!--            min-width="380">-->
<!--        </el-table-column>-->
<!--      </el-table>-->
    </container>
  </div>
</template>

<script type="text/ecmascript-6">
import Container from "../../components/Container";
import UnitTools from '@/lib/unitTools';

export default {
  name: "UnitCalc",
  info: function (){
    let list = [];
    for(let key of Object.keys(UnitTools.calcData)){
      let data = UnitTools.calcData[key];
      let cmds = [];
      for(let name of Object.keys(data.calc)){
        cmds.push({
          "type": "regex",
          "label": name,
          "match": "/[0-9\.]+"+name+"$/i",
        })
      }
      // 别名
      if (data.alias){
        for(let name of Object.keys(data.alias)){
          let alias = data.alias[name];
          if (alias instanceof Array){
            for(let aliasName of alias){
              cmds.push({
                "type": "regex",
                "label": name,
                "match": "/[0-9\.]+"+aliasName+"$/i",
              })
            }
          }else{
            cmds.push({
              "type": "regex",
              "label": name,
              "match": "/[0-9\.]+"+alias+"$/i",
            })
          }
        }
      }

      list.push({
        code: data.name,
        label: data.name,
        logo: "",
        desc: "",
        cmds: cmds
      })
    }
    return list
  },
  components: {Container},
  data() {
    return {
      results: [],

      value: 1,
      srcUnit: null,
      toUnit: '全部',

      currentUnit: {}
    }
  },
  computed:{
    unitTypes: function (){
      if (this.currentUnit.calc){
        return Object.keys(this.currentUnit.calc);
      }
      return [];
    }
  },
  mounted(){

  },
  methods: {
    callbackReturn(){
      this.$emit("callbackReturn");
    },
    init(code,value=null){
      for(let key of Object.keys(UnitTools.calcData)){
        let data = UnitTools.calcData[key];
        if (data.name === code){
          this.currentUnit = data;
          this.currentUnit.key = key;
          break;
        }
      }
      console.log(code,value);
      if (value){
        let preg = /^([0-9.]+)(.*?)$/
        let matches = value.match(preg);
        this.value = matches[1];
        this.srcUnit = matches[2].toLowerCase();
      }else{  // 设置默认
        this.value = 1;
        this.srcUnit = Object.keys(this.currentUnit.calc)[0];
      }
      if (!this.currentUnit.init[this.srcUnit]){
        this.srcUnit = this.unitAlias(this.currentUnit.key,this.srcUnit);
      }
      this.calc();
    },
    setValue(payload, type,code){
      this.init(code,payload);
    },
    async calc() {
      console.log(this.currentUnit.key, this.value, this.srcUnit, '全部');
      const loading = this.$loading({
        lock: true,
        text: '计算中',
        spinner: 'el-icon-loading',
        background: 'rgba(0, 0, 0, 0.7)'
      })
      try {
        this.results = await UnitTools.calc(this.currentUnit.key, this.value, this.srcUnit, this.toUnit);
      }finally {
        loading.close();
      }
    },

    unitAlias(name,alias){
      let data = UnitTools.calcData[name];
      for(let key of Object.keys(data.alias)){
        let keyAlias = data.alias[key];
        if (keyAlias instanceof Array){
          for(let name of keyAlias){
            if (name.toLowerCase() === alias.toLowerCase()) return key;
          }
        }else if (keyAlias.toLowerCase() === alias.toLowerCase()) return key;
      }
      return alias;
    }
  },
}
</script>

<style lang="stylus" rel="stylesheet/stylus">
.UnitCalc {
  .unit-result-item{
    border-bottom: 1px solid #eaeaea;
    margin-bottom 10px;
    padding-bottom 10px;
    margin: 5px 20px;
  }
}
</style>
