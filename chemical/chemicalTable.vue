<template>
  <div class="app-body">
    <el-table class="spcTable" :data="detailData" @sort-change="sortChange" style="width: 100%;line-height: 72px">
      <el-table-column prop="" label="" min-width="40">
        <template slot-scope="scope">
          <el-button type="text" @click="handlePreview(scope.row)">详情</el-button>
        </template>
      </el-table-column>
      <el-table-column label="Highlight" width="81">
        <template slot-scope="scope">
          <el-checkbox style="display: block;" @change="hightlightChange('plant', scope.row.id, scope.row.factory)"
            :checked="scope.row.plant == 1" label="Plant" :disabled="!tablePermission.highlight">工厂级</el-checkbox>
          <el-checkbox style="display: block;" @change="hightlightChange('area', scope.row.id, scope.row.factory)"
            :checked="scope.row.area == 1" label="Area" :disabled="!tablePermission.highlight">部门级</el-checkbox>
          <el-checkbox style="display: block;" @change="hightlightChange('highlight', scope.row.id, scope.row.factory)"
            :checked="scope.row.highlight == 1" label="Highlight" :disabled="!tablePermission.highlight">Highlight</el-checkbox>
        </template>
      </el-table-column>
      <el-table-column prop="factory" label="厂别" min-width="26">
      </el-table-column>
      <el-table-column prop="department" label="部门" sortable="custom" min-width="40">
      </el-table-column>
      <el-table-column prop="spcItemName" label="管制项目名称" min-width="250">
      </el-table-column>
      <el-table-column prop="chemicalLevel" label="药水等级" min-width="46">
      </el-table-column>
      <el-table-column prop="resourceName" label="设备编号" min-width="110">
      </el-table-column>
      <el-table-column prop="kce" label="药水参数种类" min-width="60">
      </el-table-column>
      <el-table-column prop="triggerType" label="触发类型" min-width="46">
      </el-table-column>
      <el-table-column prop="trend" label="趋势图" min-width="130">
        <template slot-scope="scope">
          <div class="trend" style="height: 120px;width:100%" :id="scope.row.id" @click="showLargeDialog(scope.row)">
          </div>
        </template>
      </el-table-column>
      <el-table-column prop="isMultiple" label="多次异常" min-width="53">
      </el-table-column>
      <el-table-column prop="peStatus" label="ME状态" min-width="53">
      </el-table-column>
      <el-table-column prop="qeStatus" label="QE状态" min-width="53">
      </el-table-column>
      <el-table-column prop="triggerStatus" label="流程状态" min-width="43">
      </el-table-column>
      <el-table-column prop="triggerTime" label="触发(工厂级/部门级)时间" min-width="80">
      </el-table-column>
      <el-table-column prop="triggerSpcTime" label="触发SPC时间" min-width="80">
      </el-table-column>
      <el-table-column prop="owner" label="责任人" min-width="41">
      </el-table-column>
      <template>
        <el-table-column
          v-for="column in dynamicList"
          :key="column.fieldName"
          :prop="column.fieldName"
          :label="column.fieldNameDesc">
        </el-table-column>
      </template>
      <el-table-column
        label="操作"
        align="center"
        width="100">
        <template slot-scope="scope">
          <el-button v-hasPermi="['spc:chemical:changePeople']" type="text" @click="changeNode(scope.row)"  :disabled="scope.row.triggerStatus === 'close'" size="small">转审</el-button>
        </template>
      </el-table-column>

    </el-table>

    <pagination v-if="queryParams.flag != 'total'" v-show="queryParams.total > 0" :total="queryParams.total"
      :page.sync="queryParams.pageNum" :limit.sync="queryParams.pageSize" @pagination="getList" />

    <Dialog ref="dialog"
      :id="id"
      :historyTable="historyTable"
      ocap="true"
      monitor="true"
      :config="config"
      :monitorTable="option" :beforeClose="beforeClose" @close="resetForm" @refreshList="getList">
    </Dialog>

    <el-dialog :visible="dialogVisible" @close="dialogVisible = false">
      <div id="bigTrendChemical" style="width:100%;height:300px;"></div>
    </el-dialog>
    <change-node-people ref="changeNodePeople" @ok="getList" :spcType="'chemical'" ></change-node-people>

  </div>
</template>
<script>
import Dialog from "@/views/spc/chemical/dialog";
import * as echarts from "echarts";
import { chemicalListPage, selectHightlight } from "@/api/qdm/chemical";
import changeNodePeople from '@/views/spc/components/changeNodePeople.vue'

export default {
  components: {
    changeNodePeople,
    Dialog
  },
  props: {
    queryParams: {
      Object,
      default: () => { }
    },
    dynamicList:{
      type: Array,
      required: true
    }
  },
  data() {
    return {
      config: {
        top: '20vh',
        width: '500px',
        center: true,
        btnTxt: ['取消', '提交'],
      },
      id: null,
      dialogVisible: false,
      detailData: [],
      option: {},
      historyTable: [
        { id: '1', operator: 'sunny', time: '2022/12/12 12:00:00', described: '触发Area' },
        { id: '2', operator: 'sunny', time: '2022/12/12 12:00:00', described: '保存ME信息' },
        { id: '3', operator: 'sunny', time: '2022/12/12 12:00:00', described: '提交' }
      ],
      tablePermission:{}
    }
  },
  mounted() {
    // this.handlePreview(row)
    const tablePermissions = {
      highlight:'chinafastprint:highlight'
    };
    for (const k in tablePermissions) {
      this.tablePermission[k] = this.$auth.hasPermi(tablePermissions[k]);
    }
  },
  methods: {
    showLargeDialog(row) {
      this.dialogVisible = true;
      const data = { ...row, id: 'bigTrendChemical' };
      setTimeout(_ => {
        this.initEchart(data);
      }, 100);
    },
    //排序
    sortChange(custom) {
      this.queryParams.orderProp = custom.prop
      this.queryParams.orderBy = custom.order
      this.getList();
    },
    hightlightChange(value, id, factory) {
      selectHightlight({ button: value, id: id, factory: factory }).then(response => {

      });
    },

    handlePreview(row) {
      this.id = row.id;
      this.open();
      // this.preview.open = true;
      // this.preview.activeName = "domain.java";
    },

    detailView(id) {
      this.id = id;
      this.open();
    },
    //弹窗所需功能 开始
    open() {
      // console.log(this.id)
      this.$refs.dialog.open(cancel => {
        // cancel();
        // console.log('点击提交按钮了')
      })
      //.then(() => {console.log(this.$refs.span)});   //这里就充分利用了open方法中返回的nextTick
    },
    beforeClose() {
      this.id = ''
    },
    changeNode(row){
      this.$refs.changeNodePeople.openDialog(row)
    },
    resetForm() {
      // 这里可以写重置表单的实现
    },
    //弹窗所需功能 结束

    //调用父组件的load
    getList() {
      this.detailData = [];
      // 深拷贝 queryParams
      let copyQueryParams = JSON.parse(JSON.stringify(this.queryParams));
      // 动态form中，input类型的动态字段是String类型，后端统一用list接收，这里使用[]数组包装参数
      this.dynamicList.forEach(field => {
        if (field.htmlType === 'input' && typeof copyQueryParams[field.fieldName] === 'string') {
          if(copyQueryParams[field.fieldName]===""){
            copyQueryParams[field.fieldName] =[];
          }else {
            copyQueryParams[field.fieldName] = [copyQueryParams[field.fieldName]];
          }
        }
      });
      chemicalListPage(copyQueryParams).then(res => {
        this.detailData = res.records;
        this.queryParams.total = res.total;
        this.getEcharts();
      });
    },
    getEcharts() {
      setTimeout(_ => {
        this.detailData.forEach((e, i) => {
          this.initEchart(e);
        });
      }, 1000);
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
      let myChart = echarts.init(document.getElementById(e.id));
      myChart.clear();
      let isBigTrend = e.id == "bigTrendChemical"
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

      let option = {
        title: { show: true, text: isBigTrend ?  e.spcItemName: '', x: 'center' },
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
    },
  }
};
</script>
<!--   height: 300px!important; -->
<style ></style>
