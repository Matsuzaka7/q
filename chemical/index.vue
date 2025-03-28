<template>
  <div  class="app-container">
    <el-form ref="queryForm" class="queryCondition" size="small" label-width="100px" :inline="true" :model="queryParams">
      <el-form-item label="开始时间" prop="beginTime">
        <el-date-picker
          v-model="queryParams.beginTime"
          type="datetime"
          format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"
          placeholder="选择日期">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="结束时间" prop="endTime">
        <el-date-picker
          v-model="queryParams.endTime"
          type="datetime"
          format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"
          placeholder="选择日期">
        </el-date-picker>
      </el-form-item>


      <el-form-item label="厂别" prop="factoryParams"  @clear="getRealFactory" >
        <select-factory  v-model="queryParams.factoryParams" multiple  @change="factoryChange"></select-factory>
      </el-form-item>
<!--      <el-form-item label="厂别" prop="factoryParams">
        <el-select v-model="queryParams.factoryParams" multiple filterable clearable placeholder="请选择">
          <el-option v-for="item in factoryList"  :key="item.factoryName" :label="item.label" :value="item.factoryName">
          </el-option>
        </el-select>
      </el-form-item>-->
<!--      <el-form-item label="部门" prop="dept">
        <el-select-v2 v-model="queryParams.dept" :options="factoryDepartmentList"  :allow-create="true"  popper-class="select-v2-popper" value-key="factoryJointId" label-key="factoryJointDepartmentName" @change="onDeptChange"  filterable clearable >
        </el-select-v2>
      </el-form-item>-->

      <el-form-item label="部门" prop="dept">
        <select-mes-dept v-model="queryParams.dept" :select-factory="this.queryParams.factoryParams" @select="onDeptChange"></select-mes-dept>
      </el-form-item>
<!--        <el-form-item label="部门" prop="dept">-->
<!--        <el-select v-model="queryParams.dept" filterable clearable collapse-tags placeholder="请选择" @change="onDeptChange">-->
<!--          <el-option v-for="(dict,index)  in factoryDepartmentList" :key="index" :label="(dict.factory?dict.factory+'-':'')+dict.deptName" :value="dict.id"></el-option>-->
<!--        </el-select>-->
<!--      </el-form-item>-->



      <el-form-item label="制程名" prop="processName">
        <el-select v-model="queryParams.processName" filterable clearable collapse-tags placeholder="请选择" @change="onProcessChange">
          <el-option v-for="(item,index) in processList" :key="index" :label="item.factory+'-'+item.resourceGroupName" :value="item.resourceGroupName"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="设备编号" prop="deviceName">
        <el-select v-model="queryParams.deviceName" filterable clearable collapse-tags placeholder="请选择">
          <el-option v-for="(item,index) in deviceList" :key="index" :label="item.factory+'-'+item.resourceName" :value="item.resourceName"></el-option>
        </el-select>
      </el-form-item>



      <!-- <el-form-item label="触发类型" prop="triggerType">
        <el-select v-model="queryParams.triggerType" filterable clearable collapse-tags placeholder="请选择">
          <el-option v-for="item in dict.type.trigger_type" :key="item.value" :label="item.label" :value="item.value"></el-option>
        </el-select>
      </el-form-item> -->

      <el-form-item label="流程状态" prop="triggerStatus">
        <el-select v-model="queryParams.triggerStatus" filterable clearable collapse-tags placeholder="请选择">
          <el-option v-for="(item, index) in dict.type.triger_status" :key="index" :label="item.label" :value="item.value"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="药水类型" prop="potionType">
        <el-select v-model="queryParams.potionType" filterable clearable collapse-tags placeholder="请选择">
          <el-option v-for="(item, index) in dict.type.chemical_type" :key="index" :label="item.label" :value="item.value"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="ME状态" prop="peStatus">
        <el-select v-model="queryParams.peStatus" filterable clearable placeholder="请选择">
          <el-option
            v-for="(dict, index) in dict.type.pe_status"
            :key="index"
            :label="dict.label"
            :value="dict.value">
          </el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="QE状态" prop="qeStatus">
        <el-select v-model="queryParams.qeStatus" filterable clearable placeholder="请选择">
          <el-option
            v-for="(dict, index) in dict.type.qe_status"
            :key="index"
            :label="dict.label"
            :value="dict.value">
          </el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="等级" prop="level">
        <el-select v-model="queryParams.level" filterable clearable collapse-tags placeholder="请选择">
          <el-option v-for="(item, index) in dict.type.high_light" :key="index" :label="item.label" :value="item.value"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="触发类型" prop="triggerTypeList">
        <el-select v-model="queryParams.triggerTypeList" multiple clearable filterable collapse-tags placeholder="请选择">
          <el-option v-for="(item, index) in dict.type.trigger_type" :key="index" :label="item.label" :value="item.value"></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="多次异常" prop="isMultiple">
        <el-select v-model="queryParams.isMultiple" clearable filterable placeholder="请选择">
          <el-option key="Y" label="Y" value="Y" />
          <el-option key="N" label="N" value="N" />
        </el-select>
      </el-form-item>
      <el-form-item label="管制项目名称" prop="spcName">
        <el-input v-model="queryParams.spcName" filterable clearable placeholder="请输入"></el-input>
      </el-form-item>
      <el-form-item label="责任人" prop="owner">
        <el-input v-model="queryParams.owner" filterable clearable placeholder="请输入"></el-input>
      </el-form-item>
      <el-form-item label="药水等级" prop="chemicalLevel">
        <el-select v-model="queryParams.chemicalLevel" clearable placeholder="请选择药水等级">
          <el-option
            v-for="dict in dict.type.cp_chemical_level"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value">
          </el-option>
        </el-select>
      </el-form-item>
      <template v-for="field in this.dynamicList">
        <el-form-item :label="field.fieldNameDesc" :prop="field.fieldName" :key="field.fieldName">
          <el-input v-if="field.htmlType === 'input'" v-model="queryParams[field.fieldName]" clearable ></el-input>
          <el-select v-else-if="field.htmlType === 'select'" v-model="queryParams[field.fieldName]" multiple clearable filterable  placeholder="请选择">
            <el-option v-for="option in field.options" :key="option.dictValue" :label="option.dictLabel" :value="option.dictValue"></el-option>
          </el-select>
        </el-form-item>
      </template>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" v-preventReClick="1000" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" v-preventReClick="1000" @click="resetQuery">重置</el-button>
        <el-button type="warning" icon="el-icon-download" size="mini" plain @click="handleExport">导出</el-button>
        <el-button icon="el-icon-chat-line-square" size="mini" v-preventReClick="1000" @click="showRule">查看规则</el-button>
      </el-form-item>
    </el-form>
    <Table ref="table"
      :queryParams="queryParams" :dynamic-list="this.dynamicList">
    </Table>
    <CommonDialog ref="commonDialog"/>
  </div>

</template>

<script>
import { selectHightlight } from "@/api/qdm/chemical";
import { download2 } from '@/utils/request';
import Table from "@/views/spc/chemical/chemicalTable";
import CommonDialog from "../components/CommonDialog";
import { save, commit, back, factoryList } from "@/api/spc/spc";
import { chemicalListPage } from "@/api/qdm/chemical";
import {getMesDeptList,getMesResourceGroupList,getMesResourceDefList} from "@/api/mes/mesdata";
import { getDynamicByCpType } from '@/api/cp/extension';
export default {
  components:{
    Table,CommonDialog
  },
  name: 'ChemicalList',
  computed:{

  },
  dicts: [
    'chemical_type','device_group','device_item','trigger_type','triger_status','high_light','qe_status','pe_status','cp_chemical_level'
  ],
  data() {
    return {
      dynamicList:[],
      tabPosition: 'left',
      activeName: 'peqedisposal',
      factoryList: [],
      departmentList: [],
      processList: [],
      deviceList: [],
      factoryDepartmentList: [],
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      clearable: true,
      totalList:[],
      joinDpetName:"",
      queryParams: {
        total: 0,
        pageNum: 1,
        pageSize: 10,
        beginTime: '',
        endTime: '',
        categor: '',
        code: '',
        type: '',
        triggerStatus: '',
        traiggerType: '',
        qeStatus:'',
        peStatus:'',
        sortType:[],
        tool:[],
        group:[],
        building: '',
        processName: '',
        deviceName: '',
        triggerType: '',
        potionType: '',
        level: '',
        triggerTypeList: [],
        isMultiple: '',
        factoryParams: [],
        dept:'',
        cp_chemical_level: '',
        attribute1:"",attribute2:"",attribute3:"",attribute4:"",attribute5:"",
        attribute6:"",attribute7:"",attribute8:"",attribute9:"",attribute10:"",
        attribute11:"",attribute12:"",attribute13:"",attribute14:"",attribute15:"",
        attribute16:"",attribute17:"",attribute18:"",attribute19:"",attribute20:"",
      },
      formXq: {
        depatmentXq: '',
        potionTypeXq: ''
      },
      preview: {
        open: false,
        isLoadData: true,
        title: "Chemical详情",
        data: {},
        activeName: "domain.java"
      },
      dateRange: [],
      dateRangeXq: [],
      product: {
        highlight: '',
        areaPlant: ''
      },
      //查询框枚举
      selForm: {
        building: [
          {value: 'X1', label: 'X1'},
          {value: 'X2', label: 'X2'},
          {value: 'X3', label: 'X3'}
        ],
        orderType: [
          {value: 'NPI', label: 'NPI'},
          {value: 'HVM', label: 'HVM'}
        ],
        parameterType: [
          {value: 'K', label: 'K'},
          {value: 'C', label: 'C'},
          {value: 'E', label: 'E'}
        ],
        parameterLevel: [
          {value: 'IPP--常规参数', label: 'IPP--常规参数'},
          {value: 'KPP--关键参数', label: 'KPP--关键参数'}
        ],
        triggerType: [
          {value: 'OOC', label: 'OOC'},
          {value: 'OOS', label: 'OOS'},
          {value: 'RULE', label: 'RULE'}
        ],
        level: [
          {value: 'Plant', label: 'plant'},
          {value: 'Area', label: 'area'},
          {value: 'Highlight', label: 'highlight'}
        ]
      },
      selXqForm: {
        depatmentXq: [
          {value: 'depart1', label: '部门一'},
          {value: 'depart2', label: '部门二'},
          {value: 'depart3', label: '部门三'}
        ],
        potionTypeXq: [
          {value: 'potion1', label: '药水一'},
          {value: 'potion2', label: '药水二'},
          {value: 'potion3', label: '药水三'}
        ],
        parameterNameXq: [
          {value: 'plant', label: 'plant'},
          {value: 'area', label: 'area'},
          {value: 'highlight', label: 'highlight'}
        ]
      },
      // centerDialogVisible: true, //弹框显隐状态
      // 模拟表头数据
      detailData:[],

      tableData:[],
      historyTable:[],

      data: [],
      data1: [],
      myChart: null,
      option : {
        title: {
          text: 'BumpAOIStepYield Sigle Defect Trend',
          left: 'center'
        },
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b} : {c}'
        },
        legend: {
          left: 'left'
        },
        dataZoom: [
          {
            type: 'slider',
            xAxisIndex: 0,
            filterMode: 'none'
          },
          {
            type: 'slider',
            yAxisIndex: 0,
            filterMode: 'none'
          },

        ],
        xAxis: {
          type: 'category',
          name: 'x',
          splitLine: { show: false },
          data: ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I']
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        yAxis: {
          type: 'log',
          name: 'y',
          minorSplitLine: {
            show: true
          }
        },
        series: [
          {
            name: 'Log2',
            type: 'line',
            data: [1, 3, 9, 27, 81, 247, 741, 2223, 6669]
          },
          {
            name: 'Log3',
            type: 'line',
            data: [1, 2, 4, 8, 16, 32, 64, 128, 256]
          }

        ]
      },
      now: new Date(1997, 9, 3),
      now1: new Date(1997, 9, 3),
      now2: new Date(1997, 9, 3),
      oneDay: 24 * 3600 * 1000,
      xData: [],
      table1: {},
      table1Data:[]
    };
  },
  filters: {},
  created() {
    this.factoryChange()
    /*    this.getFactoryList()
        this.getDepartmentList()
        this.getProcessList()
        this.getDeviceList()*/
  },

  mounted() {
    let s = new Date();
    s.setDate(s.getDate()-1);
    s.setHours('08');
    s.setMinutes('00');
    s.setSeconds('00');
    let e = new Date();
    e.setHours('08');
    e.setMinutes('00');
    e.setSeconds('00');
    this.queryParams.beginTime = s.format("yyyy-MM-dd hh:mm:ss");
    this.queryParams.endTime = e.format("yyyy-MM-dd hh:mm:ss");
    this.handleQuery();
  },
  watch: {
    'queryParams.beginTime'(newValue, oldValue) {
        if(this.queryParams.endTime && newValue) {
          this.queryParams.beginTime = new Date(newValue) > new Date(this.queryParams.endTime) ? oldValue : newValue;
        } else {
          this.queryParams.beginTime = newValue;
        }
    },
    'queryParams.endTime'(newValue, oldValue) {
        if(this.queryParams.beginTime && newValue) {
          this.queryParams.endTime = new Date(newValue) < new Date(this.queryParams.beginTime) ? oldValue : newValue;
        } else {
          this.queryParams.endTime = newValue;
        }
    },
    'queryParams.factoryParams'(newValue, oldValue) {

      this.queryParams.processName = '';
      this.queryParams.deviceName = '';
    },
    /*'queryParams.factoryParams'(newValue, oldValue){
      //制程名、设备名太多，下拉做过滤绑定
      this.factoryDepartmentList = this.departmentList.filter(o => {
        return newValue.indexOf(o.factory) > -1
      })
      // this.factoryProcessList = this.processList.filter(o => {
      //   return newValue.indexOf(o.factory) > -1
      // })
      // this.factoryDeviceList = this.deviceList.filter(o => {
      //   return newValue.indexOf(o.factory) > -1
      // })

      this.onDeptChange()
      this.onProcessChange()
      this.updateDeptValue(newValue)
    },
    'queryParams.dept'(newValue, oldValue){
      this.queryParams.processName=''
      this.queryParams.deviceName=''
      //制程名、设备数据需要重新渲染
      if(!newValue) {
        this.onDeptChange()
        this.onProcessChange()
      }
    },
    'queryParams.processName'(newValue, oldValue){
      this.queryParams.deviceName=''
    },*/
  },
  methods: {
    async getDynamicByCp(){
      getDynamicByCpType({factory:this.queryParams.factoryParams[0],cpType:"CHEMICAL"}).then(response=>{
        this.dynamicList = response.data
      })
    },
    factoryChange(){
      //工厂改动动态字段Form表单双向绑定数据清除
      for (let formKey in this.queryParams) {
        let result = formKey.replace(/\d+$/, ''); // 匹配并删除字符串末尾的一个或多个数字
        if (result=="attribute"){
          this.queryParams[formKey] ="";
        }
      }
      if (this.queryParams.factoryParams.length==1){
        this.getDynamicByCp()
      }else {
        this.dynamicList=[];
      }
    },
    initializationFactoryParams(){
      this.queryParams.factoryParams = [this.$store.state.user.currAppFactoryList[0].code];
    },
    onProcessChange(){
      this.deviceList = []
      this.getDeviceList()
    },
    getDeviceList(){
      // if(!(this.queryParams.factoryParams)){
      //   return
      // }
      this.queryParams.deviceName = '';
      getMesResourceDefList({"resourceGroupName":this.queryParams.processName,"factoryParams":this.queryParams.factoryParams}).then(response=>{
        this.deviceList = response.data
      })
    },


    getRealFactory(){
      this.queryParams.processName = '';
      this.queryParams.deviceName = '';
    },
    onDeptChange(evet){

/*      this.queryParams.dept;
      this.joinDpetName =evet.value;*/
      this.processList = []
      /*this.queryParams.processName = '';*/
      getMesResourceGroupList({"deptName":this.queryParams.dept,"factoryParams":this.queryParams.factoryParams}).then(response=>{
        this.processList = response.data
      })
    },
/*    onDeptChange(){
       this.processList = []
       this.getProcessList()
    },*/
/*    getProcessList(){
      // if(!(this.queryParams.factoryParams)){
      //   return
      // }
        getMesResourceGroupList({"deptName":this.queryParams.dept,"factoryParams":this.queryParams.factoryParams}).then(response=>{
          this.processList = response.data
        })
      },*/
/*    getDepartmentList(){
        getMesDeptList().then(response=>{
          this.departmentList = response.data
        })
      },
    getFactoryList(){
        factoryList().then(response=>{
          this.factoryList = response.data
        })
      },*/
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.dynamicList=[];
      this.resetForm("queryForm");
      this.handleQuery();
      this.factoryChange();
    },
    // 获取数据
    getList() {
      this.$refs.table.getList();
    },
    /** 导出按钮操作 */
    handleExport() {
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
      // dynamicList可能为空-空串拦截，动态字段不能以字符串传到后端，post接口会报错,
      for (let formKey in copyQueryParams) {
        let result = formKey.replace(/\d+$/, ''); // 匹配并删除字符串末尾的一个或多个数字 例attribute1截取为attribute
        if (result=="attribute" && copyQueryParams[formKey]===''){
          copyQueryParams[formKey] =[];
        }
      }
      download2(
        "/chinafastprint/chemicalInfo/exportChemical",
        {
          ...copyQueryParams,
        },
        `药水监控图原始数据_${new Date().getTime()}.xlsx`
      );
    },
    showRule(){
      this.$refs.commonDialog.show()
    },
    /*updateDeptValue(newValue) {
      //部门
      if (this.queryParams.dept) {
        let selectDept = this.departmentList.filter(t => t.factoryJointId === this.queryParams.dept)
        if (selectDept) {
          if (newValue.indexOf(selectDept[0].factory) <= -1) {
            this.queryParams.dept = ''
          }
        } else {
          this.queryParams.dept = ''
        }
      }
      //制程名
      if (this.queryParams.processName) {
        let selectProcess = this.processList.filter(t => t.factoryJointId === this.queryParams.processName)
        if (selectProcess) {
          if (newValue.indexOf(selectProcess[0].factory) <= -1) {
            this.queryParams.processName = ''
          }
        } else {
          this.queryParams.processName = ''
        }
      }
      //设备名
      if (this.queryParams.deviceName) {
        let selectDevice = this.deviceList.filter(t => t.factoryJointId === this.queryParams.deviceName)
        if (selectDevice) {
          if (newValue.indexOf(selectDevice[0].factory) <= -1) {
            this.queryParams.deviceName = ''
          }
        } else {
          this.queryParams.deviceName = ''
        }
      }

    }*/
  },
};
</script>

<!--<style scoped>
.app-container {
  padding: 6px 20px 20px 20px;
}
>>> .queryCondition .el-form-item__label {
  width: 88px;
  padding: 0 2px 0 0;
}
>>> .queryCondition .el-form-item&#45;&#45;medium * {
  line-height: 20px;
}
>>> .queryCondition .el-input__inner {
  height: 20px;
}
.queryCondition .el-button {
  padding: 0px;
}
.queryCondition .el-form-item {
  margin-bottom: 5px;
}
.queryCondition .el-form-item * {
  width: 140px;
}

.tableBox {
  margin-bottom: 20px;
}
/* 通过显隐控制input框的状态 */
.tableBox .el-input {
  display: none;
}
.tableBox .current-row .el-input {
  display: block;
}
.tableBox .current-row .el-input + span {
  display: none;
  /*color: white;*/
}
.orange{color: #FFFFFF;background-color: #FEA500;}
.green{color: #FFFFFF;background-color: #008000;}
.red{color: #FFFFFF;background-color: #FE0000;}
</style>-->

