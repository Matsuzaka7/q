<template>
  <div class="app-container">
    <el-form ref="queryForm" class="queryCondition" size="small" label-width="120px" :inline="true"
             :model="queryParams">
      <el-form-item label="开始时间" prop="startDateTime" label-width="180px">
        <el-date-picker
          :type="queryParams.dateType"
          :clearable="false"
          v-model="queryParams.startDateTime"
          :picker-options="{'firstDayOfWeek': 1}"
          type="datetime"
          format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"
          placeholder="选择日期"
          :disabled="queryParams.dateType === 'ALL'">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="结束时间" prop="endDateTime">
        <el-date-picker
          :type="queryParams.dateType"
          :clearable="false"
          v-model="queryParams.endDateTime"
          :picker-options="{'firstDayOfWeek': 1}"
          type="datetime"
          format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"
          placeholder="选择日期"
          :disabled="queryParams.dateType === 'ALL'">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="厂别" prop="factory" :rules="{required:true, message:'必填项', trigger:'blur'}">
        <select-factory v-model="queryParams.factory"  filterable clearable placeholder="请选择" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="getData">查询</el-button>
        <el-dropdown trigger="click" :hide-on-click="false" @visible-change="e => searchShow = e">
          <el-button type="text" :icon="searchShow ? 'el-icon-arrow-up' : 'el-icon-arrow-down'" @click="searchShow = !searchShow">{{ searchShow ? "收起" : "展开"}}</el-button>
          <el-dropdown-menu slot="dropdown" class="searchDrop">
            <el-dropdown-item>
              <el-form-item label="类型" prop="spcTypeList">
                <el-select v-model="queryParams.spcTypeList" multiple collapse-tags :popper-append-to-body="false" placeholder="请选择">
                  <el-option
                    v-for="dict in dict.type.cp_type"
                    :key="dict.value"
                    :label="dict.label"
                    :value="dict.value">
                  </el-option>
                </el-select>
              </el-form-item>
              <el-form-item label="产品型号类型（仅IPP）" prop="pnType" label-width="180px">
                <el-select v-model="queryParams.pnType" clearable :popper-append-to-body="false" placeholder="请选择">
                  <el-option
                    v-for="dict in dict.type.cp_pn_type"
                    :key="dict.value"
                    :label="dict.label"
                    :value="dict.value">
                  </el-option>
                  <el-option
                    label="ALL"
                    value="">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </el-form-item>

    </el-form>
    <div style="width: 100%;text-align: center;">
      <el-radio-group v-model="queryParams.dateType" @change="changeDateType">
        <el-radio label="day">日</el-radio>
        <el-radio label="week">周</el-radio>
        <el-radio label="month">月</el-radio>
        <el-radio label="ALL">ALL</el-radio>
      </el-radio-group>
    </div>
    <el-row style="margin-top: 10px;">
      <el-col :span="24" v-loading="errorChartLoading">
        <h3 style="text-align: center;font-weight: bold;color: #464646;">SPC总体统计图</h3>
        <ErrorChart height="350px" :chartData="errorChartData" @callBack="errorChartCallBack"/>
      </el-col>
    </el-row>
    <el-row>
      <el-col :span="12" v-loading="deptErrorChartloading">
        <h3 style="margin-left: 12%;color: #464646;" v-if="isShowDeptErrorChart">
          <span v-if="queryParams.dateType === 'ALL'">
            {{deptErrorChartData.startDateTime}} ~ {{deptErrorChartData.endDateTime}}
          </span>
          <span v-else>{{deptErrorChartData.queryTime}}&nbsp;&nbsp;</span>
          <strong>按部门统计图</strong>
        </h3>
        <DeptErrorChart height="350px" :chartData="deptErrorChartData" v-if="isShowDeptErrorChart" @callBack="deptErrorChartCallBack"/>
      </el-col>
      <el-col :span="12" v-loading="closeRateChartLoading">
        <CloseRateChart height="350px" :chartData="closeRateChartData" v-if="isShowCloseRateChart"/>
      </el-col>
    </el-row>
    <el-row style="margin-top: 10px;" v-if="!queryParams.spcType">
      <el-col :span="24" v-loading="spcTypeErrorChartloading">
        <h3 style="text-align: center;font-weight: bold;color: #464646;" v-if="isShowSpcTypeErrorChart">按类型统计图</h3>
        <SpcTypeErrorChart height="350px" :chartData="spcTypeErrorChartData" v-if="isShowSpcTypeErrorChart" @callBack="spcTypeErrorChartCallBack"/>
      </el-col>
    </el-row>
    <el-row style="margin-top: 10px;" v-show="isShowSpcSummaryData">
      <el-col :span="24" v-loading="spcSummaryDataloading">
        <SpcSummaryDataList ref="spcSummaryData" />
      </el-col>
    </el-row>
    <A></A>
    <B></B>
  </div>
</template>

<script>
import {
  getCloseRateStatistics,
  getDeptErrorStatistics,
  getErrorStatistics,
  getErrorStatisticsByDate,
  getSpcSummaryData,
  getSpcTypeErrorStatistics
} from "@/api/spc/spc";
import ErrorChart from "./ErrorChart";
import DeptErrorChart from "./DeptErrorChart";
import CloseRateChart from "./CloseRateChart";
import SpcTypeErrorChart from "./SpcTypeErrorChart";
import SpcSummaryDataList from "./SpcSummaryDataList";
import moment from "moment/moment";

export default {
  name: "Summary",
  dicts:['cp_type', 'cp_pn_type'],
  components: {
    ErrorChart,DeptErrorChart,CloseRateChart,SpcTypeErrorChart,SpcSummaryDataList
  },
  data() {
    return {
      searchShow: false,
      queryParams: {
        factory: '',
        spcType: '',
        spcTypeList: ['IPP', 'FM', 'CHEMICAL', 'LTS', 'EM'],
        dateType: "day",
        queryTime: undefined,
        startDateTime: undefined,
        endDateTime: undefined,
        pnType: '正式料号'
      },
      errorChartData:{},
      errorChartLoading: false,

      isShowDeptErrorChart:false,
      deptErrorChartData:{},
      deptErrorChartloading: false,

      isShowCloseRateChart:false,
      closeRateChartData:{},
      closeRateChartLoading:false,

      isShowSpcTypeErrorChart:false,
      spcTypeErrorChartData:{},
      spcTypeErrorChartloading: false,

      isShowSpcSummaryData:false,
      spcSummaryData:[],
      spcSummaryDataloading: false
    };
  },
  created() {
    this.queryParams.factory = this.$store.state.user.currAppFactoryList[0].code;
  },
  mounted() {
    this.selectDay();
  },
  methods: {
    selectDay(){
      this.queryParams.dateType = "day"
      let currentMoment = moment()
      this.queryParams.endDateTime = currentMoment.endOf('day').format('YYYY-MM-DD HH:mm:ss')
      this.queryParams.startDateTime = currentMoment.subtract(14, 'days').startOf('day').format('YYYY-MM-DD HH:mm:ss')
    },
    selectMonth(){
      this.queryParams.dateType = "month"
      let currentMoment = moment()
      this.queryParams.endDateTime = currentMoment.endOf('month').format('YYYY-MM-DD HH:mm:ss')
      this.queryParams.startDateTime = currentMoment.subtract(11, 'month').startOf('month').format('YYYY-MM-DD HH:mm:ss')
    },
    selectWeek(){
      this.queryParams.dateType = "week"
      let currentMoment = moment()
      this.queryParams.endDateTime = currentMoment.endOf('isoWeek').format('YYYY-MM-DD HH:mm:ss')
      this.queryParams.startDateTime = currentMoment.subtract(9, 'week').startOf('isoWeek').format('YYYY-MM-DD HH:mm:ss')
    },
    changeDateType(val){
      //YYYY-MM-DD HH:mm:ss
      if(val === 'day'){
        this.selectDay()
      }
      if(val === 'week'){
        this.selectWeek()
      }
      if(val === 'month'){
        this.selectMonth()
      }
      if(val === 'ALL'){
        this.queryParams.queryTime = undefined;
      }
      if(this.queryParams.factory){
        this.getData();
      }
    },
    getData(){
      this.$refs.queryForm.validate(valid => {
        if (valid) {
          this.isShowDeptErrorChart = false
          this.isShowSpcTypeErrorChart = false
          this.isShowCloseRateChart = false
          this.isShowSpcSummaryData = false
          this.$modal.loading("数据统计中，请稍候...");
          (this.queryParams.dateType === 'ALL' ?  getErrorStatistics(this.queryParams) : getErrorStatisticsByDate(this.queryParams)).then(response => {
            const data = response.data;
            this.errorChartLoading = false
            this.errorChartData = data
            const dateTimeList = data.dateTimeList;
            const today = dateTimeList[dateTimeList.length - 1];
            this.errorChartCallBack({startDateTime:today.startDateTime, endDateTime:today.endDateTime, queryTime:today.queryTime});
          }).finally(() => {
            this.$modal.closeLoading();
          });
        }
      })
    },
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getData();
    },
    async errorChartCallBack(obj){
      const queryParams = Object.assign({}, this.queryParams, obj);
      this.isShowCloseRateChart = false
      this.isShowSpcTypeErrorChart = false
      this.isShowSpcSummaryData = false
      this.deptErrorChartloading = true
      const result = await getDeptErrorStatistics(queryParams)
      this.deptErrorChartData = result.data
      if(!this.isShowDeptErrorChart){
        this.isShowDeptErrorChart = true
      }
      this.deptErrorChartloading = false
      // 按类型统计
      this.spcTypeErrorChartloading = true;
      getSpcTypeErrorStatistics(queryParams).then(result => {
        this.spcTypeErrorChartData = result.data
      }).finally(() => {
        this.spcTypeErrorChartloading = false
        if(!this.isShowSpcTypeErrorChart){
          this.isShowSpcTypeErrorChart = true
        }
      });
      // spc异常信息
      this.getSpcSummaryList(queryParams);
    },
    deptErrorChartCallBack(obj){
      const queryParams = Object.assign({}, this.queryParams, obj);
      // 关闭率
      this.closeRateChartLoading = true
      getCloseRateStatistics(queryParams).then(result => {
        this.closeRateChartData = result.data
      }).finally(() => {
        this.closeRateChartLoading = false
        if(!this.isShowCloseRateChart){
          this.isShowCloseRateChart = true
        }
      });
      // spc异常信息
      this.getSpcSummaryList(queryParams);
    },
    // SPC类型异常信息
    spcTypeErrorChartCallBack(obj){
      const queryParams = Object.assign({}, this.queryParams, obj);
      // spc异常信息
      this.getSpcSummaryList(queryParams);
    },
    // 获取spc异常信息列表
    getSpcSummaryList(queryParams){
      this.spcSummaryDataloading = true
      getSpcSummaryData(queryParams).then(result => {
        this.spcSummaryData = result.data
      }).finally(() => {
        this.spcSummaryDataloading = false
        if(!this.isShowSpcSummaryData){
          this.isShowSpcSummaryData = true
          this.$nextTick(() => {
            this.$refs.spcSummaryData.initDataList(this.spcSummaryData)
          });
        }
      });
    }
  }
};
</script>
