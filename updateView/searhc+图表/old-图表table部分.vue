<template>
  <div class="app-container" style="margin-bottom: 40px;">
    <el-table style="width: 100%" :data="dataList.slice((pageNum-1)*pageSize,pageNum*pageSize)" border>
      <el-table-column label="序号"  width="55" align="center" type="index"></el-table-column>
      <el-table-column property="spcType" width="120" align="center" label="类型" :filters="spcTypeList" :filter-method="spcTypeFilterHandler"></el-table-column>
      <el-table-column property="spcGroup" width="200" align="center" label="群组" :filters="spcGroupList" :filter-method="spcGroupFilterHandler"></el-table-column>
      <el-table-column property="highlightShow" align="center" label="Highlight" :filters="highlightList" :filter-method="highFilterHandler"></el-table-column>
      <el-table-column property="spcName" width="400" label="管制项目名称" align="center" :filters="spcNameList" :filter-method="spcNameFilterHandler"></el-table-column>
      <el-table-column property="triggerType" width="200" label="异常类型" align="center" :filters="exceptionTypeList" :filter-method="typeFilterHandler"></el-table-column>
      <el-table-column prop="trend" label="趋势图" align="center" width="130">
        <template slot-scope="scope">
          <div
            class="trend"
            style="height: 100px; width: 100%"
            :id="scope.row.id"
            @click="showLargeDialog(scope.row)"
          ></div>
        </template>
      </el-table-column>
      <el-table-column property="departmentProcessModule" width="200" align="center"  label="部门" ></el-table-column>
      <el-table-column property="owner" align="center"  width="120" label="责任人" ></el-table-column>
      <el-table-column property="status" label="状态" align="center" :filters="processStatusList" :filter-method="statusFilterHandler"></el-table-column>
      <el-table-column property="nodeStatus" label="当前节点" align="center"></el-table-column>
      <el-table-column label="触发SPC时间" width="180" align="center" prop="triggerTime">
        <template slot-scope="scope">
          <a class="link-type" @click="linkOther(scope.row)">{{ scope.row.triggerTime }}</a>
        </template>
      </el-table-column>
    </el-table>
    <pagination v-show="dataList.length > 10" :total="dataList.length" :page.sync="pageNum" :limit.sync="pageSize" :autoScroll="false" @pagination="pageChange"/>
    <IppDialog ref="IPPDialog" :id="exceptionId" pageName="ipp"></IppDialog>
    <DefectDialog ref="DEFECTDialog" :id="exceptionId" pageName="defect"></DefectDialog>
    <ChemicalDialog ref="CHEMICALDialog" :id="exceptionId" pageName="chemical"></ChemicalDialog>
    <DeviceDialog ref="DEVICEDialog" :id="exceptionId" pageName="device"></DeviceDialog>
    <FmDialog ref="FMDialog" :id="exceptionId" pageName="fm"></FmDialog>
    <LtsDialog ref="LTSDialog" :id="exceptionId" pageName="lts"></LtsDialog>
    <EnvDialog ref="EMDialog" :id="exceptionId" pageName="em"></EnvDialog>
    <el-dialog :visible="dialogVisible" @close="dialogVisible = false">
      <div id="bigTrendSpcSummary" style="width: 100%; height: 300px"></div>
    </el-dialog>
  </div>

</template>

<script>

import * as echarts from "echarts";
import IppDialog from "@/views/spc/ipp/dialog.vue";
import DefectDialog from "@/views/spc/defect/dialog.vue";
import ChemicalDialog from "@/views/spc/chemical/dialog.vue";
import DeviceDialog from "@/views/spc/device/dialog.vue";
import FmDialog from "@/views/spc/fm/dialog.vue";
import LtsDialog from "@/views/spc/lts/dialog.vue";
import EnvDialog from "@/views/spc/env/dialog.vue";

export default {
  name: "SpcSummaryDataList",
  props: {},
  components: {
    IppDialog,
    DefectDialog,
    ChemicalDialog,
    DeviceDialog,
    FmDialog,
    LtsDialog,
    EnvDialog
  },
  data() {
    return {
      pageNum: 1,
      pageSize: 10,
      exceptionId: '',
      spcTypeList:[],
      spcGroupList:[],
      highlightList:[],
      spcNameList:[],
      exceptionTypeList:[],
      processStatusList:[],
      dialogVisible: false,
      dataList:[]
    }
  },
  methods: {
    linkOther(data){
      this.exceptionId = data.id
      this.$refs[data.spcType + 'Dialog'].open();
    },
    spcTypeFilterHandler(value, row, column) {
      const property = column['property'];
      return row[property] === value;
    },
    spcGroupFilterHandler(value, row, column) {
      const property = column['property'];
      return row[property] === value;
    },
    highFilterHandler(value, row, column) {
      const property = column['property'];
      return row[property] === value;
    },
    spcNameFilterHandler(value, row, column) {
      const property = column['property'];
      return row[property] === value;
    },
    statusFilterHandler(value, row, column) {
      const property = column['property'];
      return row[property] === value;
    },
    typeFilterHandler(value, row, column) {
      const property = column['property'];
      return row[property] === value;
    },
    showLargeDialog(row) {
      this.dialogVisible = true;
      const data = { ...row, id: "bigTrendSpcSummary" };
      setTimeout((_) => {
        this.initEchart(data);
      }, 100);
    },
    initDataList(val){
      this.dataList = val;
      this.pageNum = 1;
      const spcType = [...new Set(val && val.map(item => item.spcType))];
      const spcGroup = [...new Set(val && val.map(item => item.spcGroup))];
      const spcName = [...new Set(val && val.map(item => item.spcName))];
      const highlight = [...new Set(val && val.map(item => item.highlightShow))];
      const status = [...new Set(val && val.map(item => item.status))];
      const exceptionType = [...new Set(val && val.map(item => item.triggerType))];
      this.spcTypeList = spcType.map(item => {return {text: item, value: item}});
      this.spcGroupList = spcGroup.map(item => {return {text: item, value: item}});
      this.spcNameList = spcName.map(item => {return {text: item, value: item}});
      this.highlightList = highlight.map(item => {return {text: item, value: item}});
      this.processStatusList = status.map(item => {return {text: item, value: item}});
      this.exceptionTypeList = exceptionType.map(item => {return {text: item, value: item}});
      this.$nextTick(() => {
        this.getEcharts();
      });
    },
    getEcharts() {
      if(this.dataList && this.dataList.length > 0){
        const chartList = this.dataList.slice((this.pageNum-1)*this.pageSize,this.pageNum*this.pageSize);
        chartList.forEach(e => {
          this.initEchart(e);
        });
      }
    },
    pageChange(){
      this.getEcharts();
    },
    initLine(markLineData,lineName, line, color,trendData){
      if(line){
        trendData.push(line);
        markLineData.push({
          yAxis: (line || 0) + '',
          label: {
            formatter:  lineName +"=" + line || 0,
          },
          lineStyle: { color: color },
        });
      }
    },
    countDecimalPlaces(num) {
      if (null == num) { return 0; }
      // 将数字转换成字符串
      const str = num.toString();
      // 寻找小数点
      const decimalIndex = str.indexOf('.');
      if (decimalIndex === -1) {
        // 没有小数点，返回0
        return 0;
      }
      // 截取小数点后的部分，并去除前导零
      const decimalPart = str.slice(decimalIndex + 1).replace(/^0+/, '');
      // 返回小数位数，如果去除前导零后为空字符串，返回0
      return decimalPart ? decimalPart.length : 0;
    },
    initEchart(e) {
      if (!e.trendVo) { return; }
      const domElement = document.getElementById(e.id);
      if (echarts.getInstanceByDom(domElement)) {
        echarts.dispose(domElement);
      }
      let myChart = echarts.init(domElement);
      myChart.clear();
      let isBigTrend = e.id == "bigTrendSpcSummary"
      let key = e.trendVo.lineType;
      let index = e.trendVo.time.length-1;
      let third = null;
      let areaUpB = [];
      let areaUpC = [];
      let areaLowB = [];
      let areaLowC = [];
      let upA = [];
      let upB = [];
      let upC = [];
      let lowA = [];
      let lowB = [];
      let lowC = [];
      // 清理为空数组
      let uplist = e.trendVo.upperList.filter(o=>o);
      let lowlist = e.trendVo.lowerList.filter(o=>o);
      let svlist = e.trendVo.standardList.filter(o=>o);
      e.trendVo.upperList = uplist.length == 0 ? uplist : e.trendVo.upperList;
      e.trendVo.lowerList = lowlist.length == 0 ? lowlist : e.trendVo.lowerList;
      e.trendVo.standardList = svlist.length == 0 ? svlist : e.trendVo.standardList;
      // 数据组装
      for (let inx in e.trendVo.time) {
        let upper = null;
        let low = null;
        let sv = null;
        if(e.trendVo.time[inx].indexOf(e.triggerSpcTime)>-1) { index = inx; }
        if (e.trendVo.upperList.length-1 >= inx) {
          upper = e.trendVo.upperList[inx] ? parseFloat(e.trendVo.upperList[inx]) : e.trendVo.upperList[inx];
        } else { upper = null; }
        if (upper === "" || upper === '') { upper = null; }
        if (e.trendVo.lowerList.length-1 >= inx) {
          low = e.trendVo.lowerList[inx] ? parseFloat(e.trendVo.lowerList[inx]) : e.trendVo.lowerList[inx];
        } else { low = null; }
        if (low === "" || low === '') { low = null; }
        if (e.trendVo.standardList.length-1 >= inx) {
          sv = e.trendVo.standardList[inx] ? parseFloat(e.trendVo.standardList[inx]) : e.trendVo.standardList[inx];
        } else { sv = null; }
        if (sv === "" || sv === '') { sv = null; }
        if (null != sv || (null != upper && null != low)) {
          // 没有标准线时，通过上线下线取中间值
          if (null == sv) { sv = (upper+low)/2; }
          let decimals = 0;
          // 定义abc区
          if (null != upper) {
            decimals = Math.max(this.countDecimalPlaces(upper), this.countDecimalPlaces(sv)) + 2;
            third = parseFloat(((upper - sv) / 3).toFixed(decimals));
            areaUpB.push(parseFloat((upper - third).toFixed(decimals)));
            areaUpC.push(parseFloat((sv + third).toFixed(decimals)));
            upA.push(parseFloat(((upper + areaUpB[inx])/2).toFixed(decimals)));
            upB.push(parseFloat(((areaUpB[inx] + areaUpC[inx])/2).toFixed(decimals)));
            upC.push(parseFloat(((areaUpC[inx] + sv)/2).toFixed(decimals)));
          } else if (e.trendVo.upperList.length == e.trendVo.standardList.length) {
            areaUpB.push('');
            areaUpC.push('');
          }
          if (null != low) {
            decimals = Math.max(this.countDecimalPlaces(low), this.countDecimalPlaces(sv)) + 2;
            third = parseFloat(((sv - low) / 3).toFixed(decimals));
            areaLowB.push(parseFloat((low + third).toFixed(decimals)));
            areaLowC.push(parseFloat((sv - third).toFixed(decimals)));
            lowA.push(parseFloat(((low + areaLowB[inx])/2).toFixed(decimals)));
            lowB.push(parseFloat(((areaLowB[inx] + areaLowC[inx])/2).toFixed(decimals)));
            lowC.push(parseFloat(((areaLowC[inx] + sv)/2).toFixed(decimals)));
          } else if (e.trendVo.lowerList.length == e.trendVo.standardList.length) {
            areaLowB.push('');
            areaLowC.push('');
          }
        } else {
          if (e.trendVo.upperList.length == e.trendVo.standardList.length) {
            areaUpB.push('');
            areaUpC.push('');
          }
          if (e.trendVo.lowerList.length == e.trendVo.standardList.length) {
            areaLowB.push('');
            areaLowC.push('');
          }
        }
      }

      let points = [...e.trendVo.point];
      points[index] = {
        value:e.trendVo.point[index], itemStyle:{color:'#ff0000'},
        label: {
          show: isBigTrend,
          formatter: function (param) {
            if (param.data instanceof Object) { return e.triggerType; }
            return "";
          },
        }
      }
      let lineNames = ['Cpk', 'Cp', 'Ca', 'Pp', 'Ppk', 'LOT-MR'];
      if (lineNames.indexOf(key)>-1) {
        areaUpB = [];
        areaUpC = [];
        areaLowB = [];
        areaLowC = [];
        upA = [];
        upB = [];
        upC = [];
        lowA = [];
        lowB = [];
        lowC = [];
      }
      let option = {
        title: { show: true, text: isBigTrend ?  e.spcName: '', x: 'center' },
        xAxis: { show: false, type: 'category', data: e.trendVo.time },
        yAxis: { show: false, type: 'value', scale: true },
        series: [
          {
            name: 'pointInfo', data: points, type: 'line',
            symbolSize: 2,
            itemStyle:{ normal:{ show:true, color: "#0000FF" } },
            lineStyle: { width: 0.8, },
          },
          {
            name: 'up', data: e.trendVo.upperList, type: 'line', showSymbol: false, symbol: 'none',
            itemStyle:{ normal:{ show:false, color: "#FF0000", lineStyle: { width: isBigTrend?0.8:0.5, type: (isBigTrend && e.trendVo.upperList.length==upB.length)?'dotted':'solid' } } },
            endLabel:{ show: isBigTrend, formatter: 'U'+key+' = {c}' },
            // lineStyle: { width: 0.8, },
          },
          // {
          //   name: 'areaUpA', data: e.trendVo.upperList, type: 'line', showSymbol: false, symbol: 'none',
          //   itemStyle:{ normal:{ show:false, color: "#939300", lineStyle: { width: 1.5, type: 'dotted' } } },
          //   lineStyle: { width: isBigTrend ? 0.8 : 0, },
          // },
          {
            name: 'areaUpB', data: areaUpB, type: 'line', showSymbol: false, symbol: 'none',
            itemStyle:{ normal:{ show:false, color: "#939300", lineStyle: { width: isBigTrend ? 0.8 : 0, type: 'dotted' } } },
            endLabel:{ show: isBigTrend, formatter: '{c}' },
            lineStyle: { width: isBigTrend ? 0.8 : 0, },
          },
          {
            name: 'areaUpC', data: areaUpC, type: 'line', showSymbol: false, symbol: 'none',
            itemStyle:{ normal:{ show:false, color: "#939300", lineStyle: { width: isBigTrend ? 0.8 : 0, type: 'dotted' } } },
            endLabel:{ show: isBigTrend, formatter: '{c}' },
            lineStyle: { width: isBigTrend ? 0.8 : 0, },
          },
          {
            name: 'upA', data: upA, type: 'line', showSymbol: false, symbol: 'none',
            itemStyle:{ normal:{ show:false, color: "#FF0000" } },
            endLabel:{ show: isBigTrend, formatter: 'A', color: 'red', padding: -20 },
            lineStyle: { width: 0, },
          },
          {
            name: 'upB', data: upB, type: 'line', showSymbol: false, symbol: 'none',
            itemStyle:{ normal:{ show:false, color: "#FF0000" } },
            endLabel:{ show: isBigTrend, formatter: 'B', color: 'red', padding: -20 },
            lineStyle: { width: 0, },
          },
          {
            name: 'upC', data: upC, type: 'line', showSymbol: false, symbol: 'none',
            itemStyle:{ normal:{ show:false, color: "#FF0000" } },
            endLabel:{ show: isBigTrend, formatter: 'C', color: 'red', padding: -20 },
            lineStyle: { width: 0, },
          },
          {
            name: 'low', data: e.trendVo.lowerList, type: 'line', showSymbol: false, symbol: 'none',
            itemStyle:{ normal:{ show:false, color: "#FF0000", lineStyle: { width: isBigTrend?0.8:0.5, type: (isBigTrend && e.trendVo.lowerList.length==lowB.length)?'dotted':'solid' } } },
            endLabel:{ show: isBigTrend, formatter: 'L'+key+' = {c}' },
            // lineStyle: { width: 0.8, },
          },
          // {
          //   name: 'areaLowA', data: e.trendVo.lowerList, type: 'line', showSymbol: false, symbol: 'none',
          //   itemStyle:{ normal:{ show:false, color: "#939300", lineStyle: { width: 1.5, type: 'dotted' } } },
          //   lineStyle: { width: isBigTrend ? 0.8 : 0, },
          // },
          {
            name: 'areaLowB', data: areaLowB, type: 'line', showSymbol: false, symbol: 'none',
            itemStyle:{ normal:{ show:false, color: "#939300", lineStyle: { width: isBigTrend ? 0.8 : 0, type: 'dotted' } } },
            endLabel:{ show: isBigTrend, formatter: '{c}' },
            lineStyle: { width: isBigTrend ? 0.8 : 0, },
          },
          {
            name: 'areaLowC', data: areaLowC, type: 'line', showSymbol: false, symbol: 'none',
            itemStyle:{ normal:{ show:false, color: "#939300", lineStyle: { width: isBigTrend ? 0.8 : 0, type: 'dotted' } } },
            endLabel:{ show: isBigTrend, formatter: '{c}' },
            lineStyle: { width: isBigTrend ? 0.8 : 0, },
          },
          {
            name: 'lowA', data: lowA, type: 'line', showSymbol: false, symbol: 'none',
            itemStyle:{ normal:{ show:false, color: "#FF0000" } },
            endLabel:{ show: isBigTrend, formatter: 'A', color: 'red', padding: -20 },
            lineStyle: { width: 0, },
          },
          {
            name: 'lowB', data: lowB, type: 'line', showSymbol: false, symbol: 'none',
            itemStyle:{ normal:{ show:false, color: "#FF0000" } },
            endLabel:{ show: isBigTrend, formatter: 'B', color: 'red', padding: -20 },
            lineStyle: { width: 0, },
          },
          {
            name: 'lowC', data: lowC, type: 'line', showSymbol: false, symbol: 'none',
            itemStyle:{ normal:{ show:false, color: "#FF0000" } },
            endLabel:{ show: isBigTrend, formatter: 'C', color: 'red', padding: -20 },
            lineStyle: { width: 0, },
          },
          {
            name: 'sv', data: e.trendVo.standardList, type: 'line', showSymbol: false, symbol: 'none',
            itemStyle:{ normal:{ show:false, color: "#FF0000", lineStyle: { width: isBigTrend?0.8:0.5, type: (isBigTrend && (e.trendVo.lowerList.length==lowB.length || e.trendVo.upperList.length==upB.length))?'dotted':'solid' } } },
            endLabel:{ show: isBigTrend, formatter: key+' = {c}' },
            // lineStyle: { width: 0.8, },
          },
        ],
        tooltip: {
          position: "top", // 设置触发类型为坐标轴轴线触发
          formatter: (param) => {
            if(param.seriesName != 'pointInfo') { return; }
            if (param.data instanceof Object) {
              return `<div>${param.value}</div>`;
            }
            return `<div>${param.data}</div>`;
          }, // 显示数据项的名称和值
          axisPointer: {
            type: "cross", // 设置触发的指示器类型为十字准星指示器
          },
          show: true,
        },
      };
      myChart.setOption(option);
      window.addEventListener("resize", () => { myChart.resize(); });
    }
  }
};
</script>
