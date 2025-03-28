<template>
  <div class="app-container">
    <el-form
      class="queryCondition"
      :model="queryParams"
      :rules="queryRules"
      size="small"
      label-width="110px"
      ref="queryForm"
      :inline="true"
    >
      <el-form-item label="厂别" prop="factory">
        <select-factory v-model="queryParams.factory" filterable clearable :is-init=initFac @change="onFactoryChange" />
      </el-form-item>

      <el-form-item label="时间" prop="queryTimeRange" :rules="{required:queryParams.queryParamType === 'default_query_time', message:'请选择', trigger:'change'}">
        <el-radio slot="label" v-model="queryParams.queryParamType" label="default_query_time" @change="initQueryPointsTime">时间</el-radio>
        <el-tooltip content="结束时间为计算制程能力选用规格的依据" placement="top" effect="light">
          <el-date-picker
            v-model="queryParams.queryTimeRange"
            type="datetimerange"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期" style="width: 550px;"
            value-format="yyyy-MM-dd HH:mm:ss"
            format="yyyy-MM-dd HH:mm:ss"
            @change="initQueryPointsTime">
          </el-date-picker>
        </el-tooltip>
      </el-form-item>

      <el-form-item label="加载最后" prop="queryLastNumber" :rules="{required:queryParams.queryParamType === 'default_query_count', message:'请输入', trigger:'blur'}">
        <el-radio slot="label" v-model="queryParams.queryParamType" label="default_query_count" @change="initQueryPointsTime">加载最后</el-radio>
        <el-input placeholder="请输入" type="Number" v-model="queryParams.queryLastNumber" style="width: 80%">
          <template slot="append">笔</template>
        </el-input>
      </el-form-item>

      <el-form-item label="部门" prop="departmentProcessModule">
        <el-select
          v-model="queryParams.departmentProcessModule"
          filterable
          clearable
          placeholder="请选择"
          @change="onDepartmentChange"
        >
          <el-option
            v-for="(item, index) in departmentList"
            :key="index"
            :label="item.deptName"
            :value="item.deptName"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="制程名" prop="processName">
        <el-select
          v-model="queryParams.processName"
          filterable
          clearable
          collapse-tags
          placeholder="请选择"
        >
          <el-option
            v-for="item in processList"
            :key="item.resourceGroupId"
            :label="item.resourceGroupName"
            :value="item.resourceGroupName"
          ></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="参数类型" prop="paramType">
        <el-select
          v-model="queryParams.paramType"
          filterable
          clearable
          @change="onParamTypeChange"
        >
          <el-option
            v-for="(item, index) in paramTypeList"
            :key="index"
            :label="item"
            :value="item"
          ></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="群组" prop="spcGroup">
        <el-select
          v-model="queryParams.spcGroup"
          filterable
          clearable
          @change="onSpcGroupChange"
        >
          <el-option
            v-for="(item, index) in spcGroupList"
            :key="index"
            :label="item"
            :value="item"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="档案" prop="spcFileName">
        <el-select
          v-model="queryParams.spcFileName"
          filterable
          clearable
          @change="onSpcFileNameChange"
        >
          <el-option
            v-for="(item, index) in spcFileNameList"
            :key="index"
            :label="item"
            :value="item"
          ></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button
          type="primary"
          icon="el-icon-search"
          size="mini"
          @click="handleQuery"
        >搜索</el-button
        >
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
        >导出</el-button
        >
      </el-form-item>
    </el-form>
    <el-row style="margin-bottom: 30px; margin-left: 30px">
      <el-col :span="4" style="text-align: center">
        <el-button type="primary" size="mini" @click="handleSortClick" v-hasPermi="['chemical:conf:saveChemicalConf']"
        >排序</el-button>
      </el-col>

      <el-col :span="20">
        <div class="control-line">
          <el-button
            :type="'primary'"
            plain
            icon="el-icon-s-data"
            size="mini"
            @click="taskHistogram"
            style="margin-right: 10px"
          >直方图</el-button>
          <el-checkbox v-model="hidePoint" @change="hideChange(0)">双击隐藏数据点</el-checkbox>
          <el-checkbox
            v-model="deletePoint"
            @change="hideChange(1)"
            v-has-permi="['monitor:point:delete']"
          >永久隐藏数据点</el-checkbox
          >
          <div style="margin-left: 10px; margin-top:5px; position: relative;">
            <el-tooltip effect="light">
              <div slot="content">{{chemicalTypeDetail}}</div>
              <i class="el-icon-info">类型说明</i>
            </el-tooltip>
            <el-tag type="danger" v-show="hint" style="position: absolute; left: 100%; top: 0; margin-left: 10px;">
              <i class="el-icon-error"></i>
              <span>提示：</span>
              <strong>有未勾选类型，请注意！</strong>
            </el-tag>
          </div>
        </div>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="6">
        <side-nav v-model="queryParams.spcName" :options="spcNameList" @tab-click="handleQuery" sideStyle="height: 500px" />
      </el-col>
      <el-col :span="18">
        <div>
          <div style="width: 1500px; display: flex; flex-direction: row">
            <div ref="tabChemicalMonitor" style="width: 1000px; height: 650px"></div>
            <div
              style="display: flex;flex-direction: column;padding: 100px 0 0 30px;">
              <el-button icon="el-icon-edit" @click="openTableWindow">参数列表配置</el-button>
              <el-dialog
                class="el-dialog-cus"
                :visible="dialogVisible"
                @before-close="beClose"
                :append-to-body="true"
                :close-on-click-modal="false"
                :width="dialogWidth"
                :visible.sync="showDetailList">
                <detailList ref="paramConfig" :obj = "obj" :obj2 ="obj2" @update-select-line="handleUpdateSelectLine" />
              </el-dialog>
              <div v-for="(item, index) in dict.type[this.obj.type]" :key="index" class="report-value"
                   v-show="obj[item.value]" v-if="item.condition">
                <span>{{ item.label }}:</span>
                <span>{{ formatComputedDataShow(computedData[item.value])}}</span>
              </div>
            </div>
          </div>
        </div>
      </el-col>
    </el-row>

    <chemical-dialog2 ref="PointDialog2" :id="pointDialog.id" :num="num+1" />
    <!--  直方图  -->
    <el-dialog :title="taskTitle" :visible.sync="taskOpen" :close-on-click-modal="false" width="90%" append-to-body>
      <div  style="width:100%;display: grid;place-items: center">
        <div ref="chemicalHistogram" style="height: 650px;width:820px;"/>
      </div>

    </el-dialog>
    <SortDialog
      :sort-data="spcNameList"
      @updateData="handleUpdateData"
      ref="SortDialog"
    />
  </div>
</template>

<script>
import * as echarts from "echarts";
import {getChemicalIdByJsonObjId, getChemicalList, getQueryCondition,} from "@/api/qdm/chemical";
import {saveChemicalConf} from "@/api/qdm/chemicalConf";
import {getFileNameList, getParamTypeList, getSpcGroupNameList, spcNameList} from "@/api/cp/ctrlplan";
import {getMesDeptListByFactoryNew, getMesResourceGroupList} from "@/api/mes/mesdata";
import {updateHistoryData} from "@/api/kudu/config";
import {
  computeDataForHistogram,
  computeDataForReport,
  computeGrid1ForReport,
  floorWithDecimalPlaces,
} from "@/api/fp_utils/report_utils";
import SortDialog from "./SortDialog";
import detailList from "@/views/spc/components/detailList";
import { getDynamicByCpType } from '@/api/cp/extension'

var xAxisData = [];
export default {
  name: "chemical-monitor",
  components: { ChemicalDialog2: ()=> import('./dialog.vue'), SortDialog, detailList },
  props: {
    num: {
      type: Number,
      default: 0,
    },

    id: {
      type: String,
      default: "",
    },
    spcSource: {
      type: Number,
      default: 0,
    },
  },
  dicts: ["ipp_department", "device_group", "chemical_type", "trigger_type","chemical_monitor_design_data"],
  data() {
    return {
      queryParamLoadingFlag: false,
      chemicalTypeDetail: '',
      hint: false,
      //查询spcName次数
      enterTimes: 0,
      loadingDataFlag: false,
      initFac: false,
      decimalDigits: 2,
      myBoxChart: undefined,
      optionBox: {},
      forHistogramData: {
        histogram: []
      },
      queryHistogramData: {
      },
      taskList:[],
      chemicalDicts:[],
      taskForm:{},
      taskTitle:'直方图展示',
      taskOpen:false,
      computedData: {
        cpk: undefined,
        averageValue: undefined,
        standardDeviation: undefined,
        ca: undefined,
        cp: undefined
      },
      grid1Data: {
        grid1Title: "",
        grid1Data: [],
        grid1Avg: 0,
      },
      showDetailList: false,
      showComponent: true,
      dialogVisible: false,
      dialogWidth: '50%',
      obj:{
        type:'chemical_monitor_design_data',
      },
      obj2:{
        linesName:[],
        linesNameDetail: [],
      },
      copyData: {},
      myChart: undefined,
      processList: [],
      selectLine: ["UCL", "CL", "LCL"],
      monitorData: {},
      departmentList: [],
      colorMap: {
        MU: "#8fc0dd",
        MO: "#66a274",
        PR: "#e28455",
        PO: "#8d60b1",
        SA: "#d27cca",
        IQC: "#f2cc0a",
      },
      option: {},
      spcGroupList: [],
      spcFileNameList: [],
      spcNameList: [],
      paramTypeList: [],
      minValue: "",
      maxValue: "",
      hidePoint: false,
      deletePoint: false,
      opid: undefined,
      queryParams: {
        cpType: 'CHEMICAL',
        paramType:"",
        queryParamType: null,
        queryTimeRange: [],
        queryLastNumber: null,
        beginTime: "",
        endTime: "",
        factory: "",
        departmentProcessModule: "",
        analysisType: "",
        processName: "",
        code: "",
        id: "",
        spcGroup: "",
        spcFileName: "",
        spcName: "",
        triggerTypeList: [],
        isHide:""
      },
      queryRules: {
        beginTime: [{ required: true, message: "请选择", trigger: "change" }],
        endTime: [{ required: true, message: "请选择", trigger: "change" }],
        factory: [{ required: true, message: "请选择", trigger: "change" }],
       /* departmentProcessModule: [
          { required: true, message: "请选择", trigger: "change" },
        ],*/
        analysisType: [
          { required: true, message: "请选择", trigger: "change" },
        ],
        spcGroup: [{ required: true, message: "请选择", trigger: "change" }],
        spcFileName: [{ required: true, message: "请选择", trigger: "change" }],
      },
      x: [],
      y: [],
      pointDialog: {
        id: undefined,
      },
      yList: []
    };
  },
  created() {
    var opid = this.$route.query.id;
    if (opid) {
      this.opid = opid;
      this.loadData();
    }else {
      if(this.spcSource === 1){
        this.initFac = true;
        //this.initializationFactoryParams()
      }
    }
  },
  async mounted() {
    this.initQueryTime();
    await this.getChemicalDicts();
  },
  methods: {
    initializationFactoryParams(){
      this.queryParams.factory = this.$store.state.user.currAppFactoryList[0].code;
      this.onFactoryChange();
    },
    async getChemicalDicts(){
      this.getDicts("chemical_type").then(response => {
        this.chemicalDicts = response.data
        this.chemicalTypeDetailDate();
      });
    },
    hideChange(val){
      if(val === 0){
        if(this.deletePoint){
          this.deletePoint = false;
        }
      }else if(val === 1){
        if(this.hidePoint){
          this.hidePoint = false;
        }
      }
    },
    // 直方图
    taskHistogram(){
      this.taskOpen = true;
      this.taskTitle = '直方图';
      this.taskForm = {};
      //dom完全挂载后调用
      this.$nextTick(()=>{
        this.initHistogramChart(this.queryHistogramData);
      })
    },
    initHistogramChart(data){

      //直方图 开始
      let chartDomHistogram = this.$refs.chemicalHistogram;
      this.myBoxChart = echarts.init(chartDomHistogram);
      let simpleZData = data.zData;
      let gramDataList = [];
      let y_axisDensityData = []; //正态分布频率
      let x_axisData = [];
      let y_axisData = [];
      if (simpleZData && simpleZData.length > 0) {
        simpleZData.forEach(simpleDataItem => {
          simpleDataItem.forEach(item => {
            let itemInt = Number.parseFloat(item);
            gramDataList.push(itemInt);
          })
        })
        this.forHistogramData = computeDataForHistogram(gramDataList, 4);

        for (let j = 0; j <this.forHistogramData.histogram.length ; j++) {
          let gramStr = this.forHistogramData.histogram[j].gramStr ;
          let frequency = this.forHistogramData.histogram[j].frequency ;
          let density = this.forHistogramData.histogram[j].density ;
          x_axisData.push(gramStr);
          let yyData = [];
          yyData.push(gramStr);
          yyData.push(frequency);
          let yyFrequency = {value: yyData};
          y_axisData.push(yyFrequency);

          y_axisDensityData.push(density);
        }
      }
      var colors = ['#5b9bd5', '#ed7d31', '#0070C0', '#FF3428'];
      this.optionBox = {
        title: {
          text: '直方图',
          textStyle: {
            fontSize: '16',
            color: 'rgba(112, 108, 108, 1)'

          },
          x:'center'
        },
        color: colors,
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross'
          }
        },
        grid: {
          right: '15%'
        },
        toolbox: {
          feature: {
            dataView: {
              show: false,
              readOnly: false
            },
            restore: {
              show: false
            },
            saveAsImage: {
              show: false
            }
          }
        },
        legend: {
          x: '90%', // x轴位置，'right'表示右对齐
          y: 'top', // y轴位置，'top'表示顶部对齐
          data: ['频数', '概率密度']
        },
        xAxis: [{
          type: 'category',
          axisTick: {
            alignWithLabel: true
          },
          data: x_axisData,
          axisLabel: {
            fontSize: 10,
            rotate: 45 // X轴标签倾斜 45 度
          }
        }],
        yAxis: [{
          type: 'value',
          name: '概率',
          position: 'right',
          axisLine: {
            lineStyle: {
              color: colors[1]
            }
          },
          axisLabel: {
            formatter: '{value}'
          }
        },
          {
            type: 'value',
            name: '频数',
            position: 'left',
            axisLine: {
              lineStyle: {
                color: colors[0]
              }
            },
            axisLabel: {
              formatter: '{value}'
            }
          }
        ],
        series: [{
          name: '频数',
          type: 'bar',
          barGap: '2%',
          barCategoryGap: '2%',
          yAxisIndex: 1,
          data: y_axisData
        },
          {
            name: '概率密度',
            type: 'line',
            showSymbol: false,
            smooth: true,
            yAxisIndex: 0,
            data: y_axisDensityData
          }

        ]
      };
      this.myBoxChart.setOption(this.optionBox);
      //直方图 结束
    },
    formatComputedDataShow(val) {
      if (val === undefined || val === null) {
        return "-";
      }
      return val;
    },
    handleSortClick() {
      this.$refs.SortDialog.openDialog();
    },
    async handleUpdateData(newData) {
      await saveChemicalConf({
        factory: this.queryParams.factory,
        spcGroup: this.queryParams.spcGroup,
        spcFileName: this.queryParams.spcFileName,
        spcNameList: newData,
      });
      this.spcNameList = newData;
    },
    beClose(done) {
      done();
      this.beforeClose();
      this.cancel();
    },
    // 厂别切换
    onFactoryChange() {
      this.getDepartmentList();
      this.getParamTypeList();
      this.getSpcGroupList();
      this.chemicalTypeDetailDate();
    },
    // 部门切换
    onDepartmentChange(){
      this.getSpcGroupList();
      //this.getProcessList();
    },
    // 参数类型切换
    onParamTypeChange(){
      this.getSpcGroupList();
    },
    // 群组切换
    onSpcGroupChange(){
      this.getSpcFileNameList();
    },
    // 档案切换
    onSpcFileNameChange() {
      this.getSpcNameList();
    },
    // 获取部门信息
    getDepartmentList() {
      if(!this.queryParams.factory){
        this.queryParams.departmentProcessModule = ''
        this.departmentList = [];
        return false;
      }
      getMesDeptListByFactoryNew({factory:this.queryParams.factory}).then((response) => {
        const dataList = response.data;
        this.departmentList = dataList;
        if(!dataList.find(item => item.deptName === this.queryParams.departmentProcessModule)){
          this.queryParams.departmentProcessModule = ''
        }
      });
    },
    // 获取制程名信息
    getProcessList() {
      getMesResourceGroupList({
        deptName: this.queryParams.departmentProcessModule,
        factory: this.queryParams.factory,
      }).then((response) => {
        this.processList = response.data;
      });
    },
    // 获取参数类型信息
    getParamTypeList(){
      if(!this.queryParams.factory){
        this.queryParams.paramType = ''
        this.paramTypeList = [];
        return false;
      }
      getParamTypeList({factory:this.queryParams.factory, cpType:this.queryParams.cpType}).then(response => {
        const dataList = response.data;
        this.paramTypeList = dataList;
        if(!dataList.includes(this.queryParams.paramType)){
          this.queryParams.paramType = '';
        }
      });
    },
    // 获取群组信息
    getSpcGroupList(){
      if(!this.queryParams.factory){
        this.queryParams.spcGroup = ''
        this.spcGroupList = [];
        this.getSpcFileNameList();
        return false;
      }
      getSpcGroupNameList({
        factory: this.queryParams.factory,
        cpType: this.queryParams.cpType,
        departmentProcessModule: this.queryParams.departmentProcessModule,
        paramType: this.queryParams.paramType,
        pn: this.queryParams.pn
      }).then(response => {
        const dataList = response.data;
        this.spcGroupList = dataList;
        if(!dataList.includes(this.queryParams.spcGroup)){
          this.queryParams.spcGroup = '';
        }
        this.getSpcFileNameList();
      });
    },
    // 获取档案信息
    getSpcFileNameList(){
      if(!this.queryParams.factory){
        this.queryParams.spcFileName = ''
        this.spcFileNameList = [];
        this.getSpcNameList();
        return false;
      }
      getFileNameList({
        factory: this.queryParams.factory,
        cpType: this.queryParams.cpType,
        departmentProcessModule: this.queryParams.departmentProcessModule,
        paramType: this.queryParams.paramType,
        pn: this.queryParams.pn,
        groupName: this.queryParams.spcGroup
      }).then(response => {
        const dataList = response.data;
        this.spcFileNameList = dataList;
        if(!dataList.includes(this.queryParams.spcFileName)){
          this.queryParams.spcFileName = '';
        }
        this.getSpcNameList();
      });
    },
    // 获取管制项目名称信息
    getSpcNameList() {
      //初次进入不过滤隐藏的管制项目名称
      this.enterTimes += 1;
      if(!this.queryParams.factory || !this.queryParams.spcGroup || !this.queryParams.spcFileName){
        this.queryParams.spcName = '';
        this.spcNameList = []
        return false;
      }
      if(this.enterTimes === 1){
        this.queryParams.isHide = 'N';
      }else{
        this.queryParams.isHide = 'Y';
      }
      spcNameList({
        factory: this.queryParams.factory,
        cpType: this.queryParams.cpType,
        departmentProcessModule: this.queryParams.departmentProcessModule,
        paramType: this.queryParams.paramType,
        pn: this.queryParams.pn,
        groupName: this.queryParams.spcGroup,
        controlProjectName: this.queryParams.spcName,
        fileName: this.queryParams.spcFileName,
        isHide: this.queryParams.isHide
      }).then((response) => {
        const dataList = response.data;
        this.spcNameList = dataList;
        if (dataList && dataList.length > 0){
          if(!dataList.includes(this.queryParams.spcName)){
            this.queryParams.spcName = this.spcNameList[0];
            this.searchChemicalData()
          }
        } else {
          this.queryParams.spcName = '';
        }
      });
    },
    /** 导出按钮操作 */
    handleExport() {
      this.download(
        "/chinafastprint/chemicalInfo/exportMonitor",
        {
          ...this.queryParams,
        },
        `Chemical监控图原始数据_${new Date().getTime()}.xlsx`
      );
    },
    // 搜索
    handleQuery() {
      this.$refs["queryForm"].validate((valid) => {
        if (valid) {
          this.searchChemicalData();
        }
      });
    },
    searchChemicalData() {
      this.loadingDataFlag = true
      getChemicalList(this.queryParams).then((respose) => {
        let chartDom = this.$refs.tabChemicalMonitor;
        this.queryHistogramData = respose.data;
        this.myChart = echarts.init(chartDom);
        let data = respose.data;
        this.monitorData = data;
        let pointIds = data.ids;
        this.isRateCompute = data.isRateCompute;
        this.rate = data.rate;
        this.rsdMonitor = data.rsdMonitor;
        this.obj2.linesName = data.linesName;
        this.obj2.linesNameDetail  = data.linesNameDetail;
        this.minValue = data.minValue;
        this.maxValue = data.maxValue;
        xAxisData = data.xData;
        let valueList = data.yData.map((o) => {
          return Number.parseFloat(o.value);
        });
        // debugger
        //小数点数
        this.decimalDigits = Number.parseInt(data.decimalDigits);
        //样本数
        this.sampleCount = Number.parseInt(data.sampleCount);
        this.addCondition();
        // 初始化默认勾选
        this.setObj();
        // 每次查询不同管控项目时初始化
        this.grid1Data= {grid1Title: "", grid1Data: [], grid1Avg: 0,};
        let rUcl = "R-UCL"; // 指定要移除的元素
        let rCl = "R-CL"; // 指定要移除的元素
        let rLcl = "R-LCL"; // 指定要移除的元素
        this.selectLine = this.selectLine.filter(item => item !== rUcl && item !== rCl  && item !== rLcl); // 过滤掉要移除的元素
        this.grid1Data = computeGrid1ForReport(data.sampleCount, data.zData, data.yData,data.rsdMonitor,data.decimalDigits);
        // 如果CP没有配置R-CL就不进行计算
        if(data.linesName.includes("R_UCL") ||data.linesName.includes("R_CL") || data.linesName.includes("R_LCL")){          //z是单值，y是均值
          this.selectLine.push("R-UCL");
          this.selectLine.push("R-CL");
          this.selectLine.push("R-LCL");
        }else {
          // 清空计算R-CL的代码
          this.grid1Data.grid1Data = [];
          this.grid1Data.grid1Avg = 0;
        }
        let d2 = Number.parseFloat(data.d2);
        this.computedData = computeDataForReport(
          data.zData,
          data.controlPlanLines,
          Number.parseFloat(data.computeSl),
          Number.parseFloat(data.computeUsl),
          Number.parseFloat(data.computeLsl),
          4,
          data.sampleCount,
          data.rsdMonitor,
          d2,
          data.rulesStr,
          data.unit,
          data.rate,
          data.rateUnit,
        );

        this.option = {
          animation: false,
          title: {
            text: "",
            x:'center',
            top:'6%'
          },
          grid: [
            {
              top: "10%",
              height: "38%",
            },
            {
              top: "54%",
              height: "20%",
            },
          ],
          tooltip: {
            position: "top",
            formatter: (param) => {
              let dataIndex = param.dataIndex;
              let value = param.value;
              if (param.seriesName === "DotData") {
                let zData = [];
                let rules = [];
                let flagArr = [];
                for(let key in this.option.legend.selected) {
                  if(this.option.legend.selected[key]) { flagArr.push(key); }
                }
                for(let inx in data.yData) {
                  if(flagArr.indexOf(data.yData[inx].name) > -1) {
                    zData.push(data.zData[inx]);
                    rules.push(data.rules[inx]);
                  }
                }
                if (zData[dataIndex] instanceof Array) {
                  let detailData = zData[dataIndex].sort((a, b) => a - b);
                  if (param.data.isErrorPoint) {
                    return `RULE：${
                        data.rules[dataIndex]
                    }<br/>AVG：${value}<br/>MAX：${
                      detailData[detailData.length - 1]
                    }<br/>MIN：${detailData[0]}`;
                  }
                  return `AVG：${value}<br/>MAX：${
                    detailData[detailData.length - 1]
                  }<br/>MIN：${detailData[0]}`;
                } else {
                  return `暂无数据`;
                }
              }
              return value;
            }, // 显示数据项的名称和值
            axisPointer: {
              type: "cross", // 设置触发的指示器类型为十字准星指示器
            },
          },
          xAxis: [
            {
              type: "category",
              boundaryGap: false,
              show: false,
              data: [],
              gridIndex: 0,
            },
            {
              type: "category",
              boundaryGap: false,
              splitLine: {
                show: false,
              },
              axisLabel: {
                rotate: 70,
                interval: 0,
              },
              data: [],
              gridIndex: 1,
            },
            {
              type: "category",
              boundaryGap: false,
              show: false,
              data: [],
              gridIndex: 0,
            },
          ],
          yAxis: [
            {
              scale:true,
              type: "value",
              name: this.grid1Data.grid1Title,
              nameGap: 50,
              boundaryGap: ["10%", "10%"],
              nameLocation: "center",
              splitLine: {
                show: false,
              },
              axisLine: { show: true },
              gridIndex: 0,
            },
            {
              type: "value",
              name: this.grid1Data.grid1Title2,
              nameGap: 50,
              nameLocation: "center",
              boundaryGap: [0, "100%"],
              splitLine: {
                show: false,
              },
              axisLine: { show: true },
              gridIndex: 1,
            },
          ],
          legend: {
            left: "8%",
            //"MU", "MO", "PR", "PO", "SA", "IQC"
            data: [],
            selected:{"MU":false, "MO":false, "IQC":false, "PR":true, "PO":true, "SA":true}
          },
          dataZoom: [
            {
              type: "slider",
              xAxisIndex: [0, 1],
              start: 0,
              end: 100,
            },
            {
              type: "slider",
              yAxisIndex: 0,
              start: 0,
              end: 100,
            },
            {
              type: "slider",
              yAxisIndex: 1,
              start: 0,
              end: 100,
            },
          ],
          series: [
            {
              name: "DotData",
              data: [],
              type: "line",
              lineStyle: {
                type: "solid",
                width: 0.5,
                color: "#0000FF",
              },
              symbolSize: 4,
              markLine: {
                silent: true,
                symbol: ["none", "none"],
                data: [],
              },
            },
            {
              name: "UCL",
              xAxisIndex: 0,
              type: "line",
              step: "UCL",
              showSymbol: true,
              symbol: "circle",
              symbolSize: 1,
              itemStyle:{
                normal:{
                  show:true,
                  color: "#ffd500"
                }
              },
              lineStyle: {
                type: "dashed",
                width: 0.8,
              },
              endLabel:{
                show: true,
                formatter: 'UCL = {c}'
              },
              data: [],
            },
            {
              name: "CL",
              xAxisIndex: 0,
              type: "line",
              step: "CL",
              showSymbol: true,
              symbol: "circle",
              symbolSize: 1,
              lineStyle: {
                color: "blue",
                type: "dashed",
                width: 0.8,
              },
              endLabel:{
                show: true,
                formatter: 'CL = {c}'
              },
              data: [],
            },
            {
              name: "LCL",
              xAxisIndex: 0,
              type: "line",
              step: "LCL",
              showSymbol: true,
              symbol: "circle",
              symbolSize: 1,
              itemStyle:{
                normal:{
                  show:true,
                  color: "#ffd500"
                }
              },
              lineStyle: {
                type: "dashed",
                width: 0.8,
              },
              endLabel:{
                show: true,
                formatter: 'LCL = {c}'
              },
              data: [],
            },
            {
              name: "USL",
              xAxisIndex: 0,
              type: "line",
              step: "USL",
              showSymbol: true,
              symbol: "circle",
              symbolSize: 1,
              itemStyle:{
                normal:{
                  show:true,
                  color: "#ff0000"
                }
              },
              lineStyle: {
                type: "dashed",
                width: 0.8,
              },
              endLabel:{
                show: true,
                formatter: 'USL = {c}'
              },
              data: [],
            },
            {
              name: "SL",
              xAxisIndex: 0,
              type: "line",
              step: "SL",
              showSymbol: true,
              symbol: "circle",
              symbolSize: 1,
              lineStyle: {
                color: "green",
                type: "dashed",
                width: 0.8,
              },
              endLabel:{
                show: true,
                formatter: 'SL = {c}'
              },
              data: [],
            },
            {
              name: "LSL",
              xAxisIndex: 0,
              type: "line",
              step: "LSL",
              showSymbol: true,
              symbol: "circle",
              symbolSize: 1,
              itemStyle:{
                normal:{
                  show:true,
                  color: "#ff0000"
                }
              },
              lineStyle: {
                type: "dashed",
                width: 0.8,
              },
              endLabel:{
                show: true,
                formatter: 'LSL = {c}'
              },
              data: [],
            },
            {
              name: "grid1Data",
              type: "line",
              xAxisIndex: 1,
              yAxisIndex: 1,
              showSymbol: true,
              symbol: "circle",
              symbolSize: 4,
              lineStyle: {
                type: "solid",
                width: 0.5,
              },
              color: "#308200",
              endLabel: {
                show: false,
                formatter: "",
              },
              data: this.grid1Data.grid1Data,
            },
            {
              name: "R-UCL",
              xAxisIndex: 1,
              yAxisIndex: 1,
              type: "line",
              step: "R-UCL",
              showSymbol: true,
              symbol: "circle",
              symbolSize: 1,
              itemStyle:{
                normal:{
                  show:true,
                  color: "#ff0000"
                }
              },
              lineStyle: {
                type: "dashed",
                width: 0.8,
              },
              endLabel:{
                show: true,
                formatter: 'R-UCL = {c}'
              },
              data: []
            },
            {
              name: "R-CL",
              xAxisIndex: 1,
              yAxisIndex: 1,
              type: "line",
              step: "R-CL",
              showSymbol: true,
              symbol: "circle",
              symbolSize: 1,
              lineStyle: {
                type: "dashed",
                width: 0.8,
              },
              endLabel:{
                show: true,
                formatter: 'R-CL = {c}'
              },
              data: []
            },
            {
              name: "R-LCL",
              xAxisIndex: 1,
              yAxisIndex: 1,
              type: "line",
              step: "R-LCL",
              showSymbol: true,
              symbol: "circle",
              symbolSize: 1,
              itemStyle:{
                normal:{
                  show:true,
                  color: "#ff0000"
                }
              },
              lineStyle: {
                type: "dashed",
                width: 0.8,
              },
              endLabel:{
                show: true,
                formatter: 'R-LCL = {c}'
              },
              data: []
            },{
              name: "UAL",
              xAxisIndex: 2,
              type: "line",
              step: "UAL",
              showSymbol: true,
              symbol: "cirALe",
              symbolSize: 1,
              itemStyle:{
                normal:{
                  show:true,
                  color: "#ffd500"
                }
              },
              lineStyle: {
                type: "dashed",
                width: 0.8,
              },
              endLabel:{
                show: true,
                formatter: 'UAL = {c}'
              },
              data: [],
            },
            {
              name: "AL",
              xAxisIndex: 2,
              type: "line",
              step: "AL",
              showSymbol: true,
              symbol: "cirALe",
              symbolSize: 1,
              lineStyle: {
                color: "blue",
                type: "dashed",
                width: 0.8,
              },
              endLabel:{
                show: true,
                formatter: 'AL = {c}'
              },
              data: [],
            },
            {
              name: "LAL",
              xAxisIndex: 2,
              type: "line",
              step: "LAL",
              showSymbol: true,
              symbol: "cirALe",
              symbolSize: 1,
              itemStyle:{
                normal:{
                  show:true,
                  color: "#ffd500"
                }
              },
              lineStyle: {
                type: "dashed",
                width: 0.8,
              },
              endLabel:{
                show: true,
                formatter: 'LAL = {c}'
              },
              data: [],
            },{
              name: 'UA',
              type: 'line',
              step: "UA",
              symbolSize: 0,
              lineStyle: {
                color: "#D9D9A7",
                width: 0,
                type: "dashed",
              },
              label: {
                show: true,
                position: 'top',
                offset: [20, 25],
                color: '#ff0000',
                formatter: function(params){
                  return params.dataIndex === 0 ? 'A' : '';
                }
              }
            },
            {
              name: 'UB',
              type: 'line',
              step: "UB",
              symbolSize:0,
              lineStyle:{
                color:"#D9D9A7",
                width:1,
                type:"dashed"
              },
              label: {
                show: true,
                position: 'top',
                offset: [20, 25],
                color: '#ff0000',
                formatter: function(params){
                  return params.dataIndex === 0 ? 'B' : '';
                }
              }
            },
            {
              name: 'UC',
              type: 'line',
              step: "UC",
              symbolSize:0,
              lineStyle:{
                color:"#D9D9A7",
                width:1,
                type:"dashed"
              },
              label: {
                show: true,
                position: 'top',
                offset: [20, 25],
                color: '#ff0000',
                formatter: function(params){
                  return params.dataIndex === 0 ? 'C' : '';
                }
              }
            },
            {
              name: 'LC',
              type: 'line',
              step: "LC",
              symbolSize:0,
              lineStyle:{
                color:"#D9D9A7",
                width:1,
                type:"dashed"
              },
              label: {
                show: true,
                position: 'bottom',
                offset: [20, -25],
                color: '#ff0000',
                formatter: function(params){
                  return params.dataIndex === 0 ? 'C' : '';
                }
              }
            },
            {
              name: 'LB',
              type: 'line',
              step: "LB",
              symbolSize:0,
              lineStyle:{
                color:"#D9D9A7",
                width:1,
                type:"dashed"
              },
              label: {
                show: true,
                position: 'bottom',
                offset: [20, -25],
                color: '#ff0000',
                formatter: function(params){
                  return params.dataIndex === 0 ? 'B' : '';
                }
              }
            },
            {
              name: 'LA',
              type: 'line',
              step: "LA",
              symbolSize:0,
              lineStyle:{
                color:"#D9D9A7",
                width:0,
                type:"dashed"
              },
              label: {
                show: true,
                position: 'bottom',
                offset: [20, -25],
                color: '#ff0000',
                formatter: function(params){
                  return params.dataIndex === 0 ? 'A' : '';
                }
              }
            }
          ],
        };
        this.option.title.text = this.sampleCount > 1 ? ( this.rsdMonitor === "Xbar-S" ? this.queryParams.spcName+"的Xbar-S（均值标准差）控制图": this.queryParams.spcName+"的Xbar-R（均值极差）控制图")  : this.queryParams.spcName +"的X-MR（单值移动极差）控制图" ,
        this.option.title.subtext = data.controlPlanInfo && data.controlPlanInfo.remark ? data.controlPlanInfo.remark : '';
        this.option.title.subtextStyle = { color: '#696969' }
        this.option.xAxis[0].data = data.xData;
        this.option.xAxis[1].data = data.xData;
        // debugger
        //设置颜色
        if (data.symbolList && data.symbolList.length > 0) {
          let legendData = [];
          data.symbolList.forEach((o) => {
            o.itemStyle = {
              //"color": this.colorMap[o.name]
              color: "#0000FF",
            };
            legendData.push(o.name);
          });
          this.option.legend.data = legendData
        }
        this.option.series = this.option.series.concat(data.symbolList);

        /* if (data.yData && data.yData.length > 0) {
          data.yData.forEach(o => {
            o.itemStyle = {
              "color": this.colorMap[o.name]
            }
          })
        } */
        this.markExceptionPoint(data);
        this.option.series[0].data = data.yData;
        this.copyData = this.deepCopy(data);
        // this.option.yAxis.min = function (value) {
        //   return value.min;
        // };
        // this.option.yAxis.max = function (value) {
        //   return value.max;
        // };
        this.handleMarkLine();
        this.myChart.clear();
        this.myChart.setOption(this.option);
        this.myChart.off("click");
        this.myChart.on("click", async (params) => {
          if (this.num > 2) {
            return false;
          }
          // 开启双击隐藏点，禁用单击功能
          if (this.hidePoint) {
            return false;
          }
          if (this.deletePoint) {
            return false;
          }
          if (params.seriesName == "DotData" && params.data.isErrorPoint) {
            const result = await getChemicalIdByJsonObjId(
              data.exceptionPointIds[params.dataIndex]
            );
            if (result.data) {
              this.pointDialog.id = result.data;
              this.$refs.PointDialog2.open();
            }
          }
        });
        let optionFlag = JSON.parse(JSON.stringify(this.option));
        // 图例点击
        this.myChart.off("legendselectchanged");
        this.myChart.on("legendselectchanged", (params) => {
          // 有任何一个未勾选都展示
          this.hint = Object.values(params.selected).includes(false);
          // this.option.series[0].data = data.yData.filter(
          //   (item) => params.selected[item.name]
          // );
          let newXdata = [];
          let newRules = [];
          let exceptionPointIds = [];
          let inxArr = optionFlag.series[0].data.filter((item) => params.selected[item.name]).map((item)=>optionFlag.series[0].data.indexOf(item));
          for(let inx in optionFlag.series) {
            if(!optionFlag.series[inx].data) { continue; }
            this.option.series[inx].data = optionFlag.series[inx].data.filter((item, i) => inxArr.indexOf(i)>-1);
          }
          for(let key of inxArr) {
              newXdata.push(optionFlag.xAxis[0].data[key]);
              newRules.push(this.copyData.rules[key]);
              exceptionPointIds.push(this.copyData.exceptionPointIds[key]);
            }
          let valueList = this.option.series[0].data.map((o) => {
            return Number.parseFloat(o.value);
          });
          this.computedData = computeDataForReport(
            data.zData,
            data.controlPlanLines,
            Number.parseFloat(data.computeSl),
            Number.parseFloat(data.computeUsl),
            Number.parseFloat(data.computeLsl),
            4,
            data.sampleCount,
            data.rsdMonitor,
            d2,
            data.rulesStr,
            data.unit,
            data.rate,
            data.rateUnit,
          );

          this.$set(this.computedData, 'chemicalLevel', data.chemicalLevel);

          this.option.series.filter((o) => {
            return o.name === "grid1Data";
          })[0].data = this.grid1Data.grid1Data;

          let xTypeArr = [];
          for (let i = 0; i < data.yData.length; i++) {
            xTypeArr.push(data.yData[i].name);
          }
          let seletedObj = params.selected;
          let map = new Map();
          for (let key in seletedObj) {
            if (seletedObj.hasOwnProperty(key)) {
              map.set(key, seletedObj[key]);
            }
          }

          // let newXdata = [];
          // for (let i = 0; i < data.xData.length; i++) {
          //   let type = xTypeArr[i];
          //   if (map.get(type)) {
          //     newXdata.push(data.xData[i]);
          //   }
          // }
          // let newRules = [];
          // for (let i = 0; i < this.copyData.rules.length; i++) {
          //   let type = xTypeArr[i];
          //   if (map.get(type)) {
          //     newRules.push(this.copyData.rules[i]);
          //   }
          // }
          data.rules = newRules;
          // let exceptionPointIds = [];
          // for (let i = 0; i < this.copyData.exceptionPointIds.length; i++) {
          //   let type = xTypeArr[i];
          //   if (map.get(type)) {
          //     exceptionPointIds.push(this.copyData.exceptionPointIds[i]);
          //   }
          // }
          data.exceptionPointIds = exceptionPointIds;
          this.option.xAxis.data = newXdata;
          this.option.xAxis[0].data = newXdata;
          this.option.xAxis[1].data = newXdata;
          this.option.legend.selected = params.selected;
          this.myChart.clear();
          this.myChart.setOption(this.option);
        });
        this.myChart.dispatchAction({
          type: 'legendSelect',
          name: 'MU'
        });
        this.myChart.dispatchAction({
          type: 'legendToggleSelect',
          name: 'MU'
        });
        // 双击隐藏点
        this.myChart.off("dblclick");
        this.myChart.on("dblclick", "series.line", (params) => {
          if (this.num > 2) {
            return false;
          }
          // 异常点不能隐藏
          // if(params.data && params.data.isErrorPoint){
          //   this.$modal.msgWarning('异常点不能隐藏');
          //   return false;
          // }
          if (params.seriesIndex === 0) {
            this.hideDataPoint(pointIds, params.dataIndex);
          }
        });
        this.$set(this.computedData, 'chemicalLevel', data.chemicalLevel);
        this.loadingDataFlag = false
      })
        .catch(() => {
          this.loadingDataFlag = false
        })
    },

    deepCopy(obj) {
      return JSON.parse(JSON.stringify(obj));
    },
    // 隐藏数据点
    hideDataPoint(pointIds, dataIndex) {
      if (this.hidePoint) {
        let clList = ["USL", "SL", "LSL", "UCL", "CL", "LCL","UAL", "AL", "LAL", "R-UCL", "R-CL", "R-LCL","UA", "UB", "UC","LA", "LB", "LC"];
        for (let val of clList) {
          let clTypeName = val.toLowerCase().replace("-", "_");
          if (this.monitorData[clTypeName + "List"].length > 0) {
            this.monitorData[clTypeName + "List"].splice(dataIndex, 1);
          }
        }
        this.option.series[0].data.splice(dataIndex, 1);
        this.option.xAxis[0].data.splice(dataIndex, 1);
        this.option.xAxis[1].data.splice(dataIndex, 1);
        let data = this.monitorData;
        let valueList = this.option.series[0].data.map((o) => {
          return Number.parseFloat(o.value);
        });
        let d2 = Number.parseFloat(data.d2);
        let newData = computeDataForReport(
          data.zData,
          data.controlPlanLines,
          Number.parseFloat(data.computeSl),
          Number.parseFloat(data.computeUsl),
          Number.parseFloat(data.computeLsl),
          4,
          data.sampleCount,
          data.rsdMonitor,
          d2,
          data.rulesStr,
          data.unit,
          data.rate,
          data.rateUnit,
        );

        this.computedData = { ...this.computedData, ...Object.assign({}, newData) };

        this.option.series.filter((o) => {
          return o.name === "grid1Data";
        })[0].data = this.grid1Data.grid1Data;
        this.handleMarkLine();
        this.myChart.clear()
        this.myChart.setOption(this.option);
      }
      if (this.deletePoint) {
        updateHistoryData({
          idList: pointIds.splice(dataIndex, 1),
          type: "CHEMICAL",
          status: 0,
        }).then((res) => {
          console.log("操作隐藏成功");
          this.searchChemicalData();
        });
      }
    },
    chemicalTypeDetailDate(){
      // 初始化类型详情
      let chemicalList = this.chemicalDicts.filter(a=> this.queryParams.factory === a.factory);
      let chemicalTypeDetails = "" ;
      for (let i = 0; i < chemicalList.length; i++) {
        let chemical = chemicalList[i];
        chemicalTypeDetails += chemical.dictLabel + ':' + chemical.remark + ";";
       }
      this.chemicalTypeDetail = chemicalTypeDetails;
    },
    loadData() {
      this.enterTimes = 0;
      getQueryCondition({ id: this.opid ? this.opid : this.id }).then(
        async (response) => {
          this.queryParams = response.data;
          await this.initQueryTime();
          this.onFactoryChange();
          this.searchChemicalData();
          //this.chemicalTypeDetailDate();
        }
      );
    },
    addCondition(){
      //初始化时把其他的判断加上
      for (let i = 0; i < this.dict.type[this.obj.type].length; i++) {
        let deviceList = this.dict.type[this.obj.type][i];
        if(deviceList.label === "均值极差"){
          this.$set(this.dict.type[this.obj.type][i], 'condition', this.rsdMonitor === 'Xbar-R');
        }else if(deviceList.label === "均值移动极差"){
          this.$set(this.dict.type[this.obj.type][i], 'condition', this.sampleCount === 1);
        }else if(deviceList.label === "均值标准差"){
          this.$set(this.dict.type[this.obj.type][i], 'condition', this.rsdMonitor === 'Xbar-S');
        }else {
          this.$set(this.dict.type[this.obj.type][i], 'condition', true);
        }
      }
    },
    setObj(){
      //初始化时把其他的判断加上
      for (let i = 0; i < this.dict.type[this.obj.type].length; i++) {
        let ippList = this.dict.type[this.obj.type][i];
        if(ippList.raw.cssClass === "N"){
          // 如果初始化为N就默认不展示
          this.$set(this.obj, ippList.value, false);
        }else {
          this.$set(this.obj, ippList.value, true);
        }
      }
    },
    async initQueryTime() {
      const dictData = await this.getDicts('default_query_parameters');
      const defaultQueryParameter = dictData.data;
      const defaultQueryTime = defaultQueryParameter.find(item => item.dictLabel === 'default_query_time');
      const defaultQueryCount = defaultQueryParameter.find(item => item.dictLabel === 'default_query_count');
      this.queryParams.queryParamType = defaultQueryParameter[0].dictLabel;
      const preToday = Number(defaultQueryTime.dictValue || "1");
      let s = new Date();
      s.setDate(s.getDate() - preToday);
      s.setHours("00");
      s.setMinutes("00");
      s.setSeconds("00");
      let e = new Date();
      this.queryParams.queryTimeRange = [s.format("yyyy-MM-dd hh:mm:ss"), e.format("yyyy-MM-dd hh:mm:ss")];
      this.queryParams.queryLastNumber = defaultQueryCount && Number(defaultQueryCount.dictValue) || null;
      this.initQueryPointsTime();
    },
    initQueryPointsTime(){
      if(this.queryParams.queryParamType === 'default_query_time'){
        this.queryParams.beginTime = this.queryParams.queryTimeRange[0];
        this.queryParams.endTime = this.queryParams.queryTimeRange[1];
      } else {
        this.queryParams.beginTime = null;
        this.queryParams.endTime = null;
      }
    },
    // 标记异常点红色
    markExceptionPoint(data) {
      if (data.yData && data.yData.length > 0) {
        for (let i = 0; i < data.yData.length; i++) {
          if (data.exceptionList[i]) {
            data.yData[i].itemStyle = { color: "#ff0000" };
            data.yData[i].isErrorPoint = true;
          }
        }
      }
    },
    openTableWindow(){
      this.showDetailList = true;
      this.$nextTick(() => {
        this.$refs.paramConfig.initData();
      });
    },
    // 更新父组件中的 selectLine 数组
    handleUpdateSelectLine(value) {
      this.selectLine = value;
      //控制线变更
      this.handleMarkLine();
    },
    // 控制线勾选
    handleMarkLine(e) {
      let markLineData = [];
      let clList = ["USL", "SL", "LSL", "UCL", "CL", "LCL","UAL", "AL", "LAL","R-UCL", "R-CL", "R-LCL","UA", "UB", "UC","LA", "LB", "LC"];
      let historyCL = { USL: [], SL: [], LSL: [], UCL: [], CL: [], LCL: [],UAL: [], AL: [], LAL: [], 'R-UCL':[], 'R-CL':[], 'R-LCL':[],UA:[], UB:[], UC:[],LA:[], LB:[], LC:[]};
      if(this.selectLine.includes('ABC区')){
        // 如果包含abc区就代表需要展示最多6条线
        this.selectLine.push("UA", "UB", "UC","LA", "LB", "LC");
      }
      for (let val of this.selectLine) {
        let clLineName = val.toLowerCase().replace("-", "_");
        let lineVal = this.monitorData[clLineName];
        if (lineVal) {
          // 同一个类型有多条数据，说明有历史控制线
          if (
            clList.indexOf(val) !== -1 &&
            this.monitorData[clLineName + "List"] && this.monitorData[clLineName + "List"].length > 1
          ) {
            historyCL[val] = this.monitorData[clLineName + "List"];
          } else {
            historyCL[val] = Array(this.monitorData.xData.length).fill(lineVal);
          }
        }else{
          if(val === 'R-CL' && this.grid1Data){
            let grid1Avg = floorWithDecimalPlaces(
              this.grid1Data.grid1Avg,
              this.decimalDigits
            )
            historyCL[val] = Array(this.monitorData.xData.length).fill(grid1Avg);
          }
        }
      }
      let autoCL = ["autoUcl", "autoCl", "autoLcl"];
      for (let cl of autoCL) {
        if (
          this.selectLine.indexOf(cl.replace("auto", "").toUpperCase()) !== -1
        ) {
          let lineVal = this.monitorData[cl];
          if (lineVal) {
            markLineData.push({
              yAxis: floorWithDecimalPlaces(lineVal,this.decimalDigits).toString(),
              label: {
                formatter: cl + " = {c}",
              },
              lineStyle: {
                color: "#939300",
              },
            });
          }
        }
      }
      // 历史控制线
      this.option.series.forEach((item) => {
        if (clList.indexOf(item.name) !== -1) {
          item.data = historyCL[item.name];
        }
      });
      this.option.series[0].markLine.data = markLineData;
      this.myChart.clear()
      this.myChart.setOption(this.option);
    },
    resetForm() {},
  },
};
</script>
<style scoped>
.report-value {
  text-align: left;
  background-color: #f2f2f2;
  color: #1e1e1e;
  width: 150px;
}
</style>

