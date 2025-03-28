<template>
  <div class="app-container">
    <el-form ref="queryForm" class="queryCondition" size="small" label-width="100px" :inline="true" :model="queryParams">
      <el-form-item label="开始时间" prop="beginTime">
        <el-date-picker v-model="queryParams.beginTime" type="datetime" value-format="yyyy-MM-dd HH:mm:ss"
                        format="yyyy-MM-dd HH:mm:ss" placeholder="选择日期">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="结束时间" prop="endTime">
        <el-date-picker v-model="queryParams.endTime" type="datetime" value-format="yyyy-MM-dd HH:mm:ss"
                        format="yyyy-MM-dd HH:mm:ss" placeholder="选择日期">
        </el-date-picker>
      </el-form-item>

      <el-form-item label="厂别" prop="factoryParams">
        <select-factory  v-model="queryParams.factoryParams" multiple @change="factoryChange"></select-factory>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" v-preventReClick="1000"
                   @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" v-preventReClick="1000" @click="resetQuery">重置</el-button>
        <el-dropdown trigger="click" :hide-on-click="false" @visible-change="e => searchShow = e">
          <el-button type="text" :icon="searchShow ? 'el-icon-arrow-up' : 'el-icon-arrow-down'" @click="searchShow = !searchShow">{{ searchShow ? "收起" : "展开"}}</el-button>
          <el-dropdown-menu slot="dropdown" class="searchDrop">
            <el-dropdown-item>
              <el-form-item label="参数类型" prop="category">
                <el-select v-model="queryParams.category" filterable clearable :popper-append-to-body="false" placeholder="请选择">
                  <el-option v-for="(dict, index) in dict.type.cp_param_type" :key="dict.value" :label="dict.label" :value="dict.value">
                  </el-option>
                </el-select>
              </el-form-item>

              <el-form-item label="参数等级" prop="variety">
                <el-select v-model="queryParams.variety" filterable clearable :popper-append-to-body="false" placeholder="请选择">
                  <el-option v-for="(dict, index) in dict.type.cp_param_level" :key="index" :label="dict.label" :value="dict.value">
                  </el-option>
                </el-select>
              </el-form-item>

              <el-form-item label="部门" prop="dept">
                <select-mes-dept v-model="queryParams.dept" :popper-append-to-body="false" :select-factory="this.queryParams.factoryParams"></select-mes-dept>
              </el-form-item>

              <el-form-item label="产品族" prop="productFamily">
                <el-select v-model="queryParams.productFamily" filterable clearable :popper-append-to-body="false" placeholder="请选择">
                  <el-option v-for="(dict, index) in dict.type.product_category" :key="index" :label="dict.label" :value="dict.value">
                  </el-option>
                </el-select>
              </el-form-item>

              <el-form-item label="产品型号" prop="pn">
                <el-input v-model="queryParams.pn" filterable clearable placeholder="请输入"></el-input>
              </el-form-item>

              <el-form-item label="ME状态" prop="peStatus">
                <el-select v-model="queryParams.peStatus" filterable clearable :popper-append-to-body="false" placeholder="请选择">
                  <el-option v-for="(dict, index) in dict.type.pe_status" :key="index" :label="dict.label" :value="dict.value">
                  </el-option>
                </el-select>
              </el-form-item>

              <el-form-item label="QE状态" prop="qeStatus">
                <el-select v-model="queryParams.qeStatus" filterable clearable :popper-append-to-body="false" placeholder="请选择">
                  <el-option v-for="(dict, index) in dict.type.qe_status" :key="index" :label="dict.label" :value="dict.value">
                  </el-option>
                </el-select>
              </el-form-item>

              <!-- <el-form-item label="触发类型" prop="triggerType">
                <el-select v-model="queryParams.triggerType" filterable clearable :popper-append-to-body="false" placeholder="请选择">
                  <el-option v-for="dict in dict.type.trigger_type" :key="dict.value" :label="dict.label" :value="dict.value">
                  </el-option>
                </el-select>
              </el-form-item> -->

              <el-form-item label="等级" prop="level">
                <el-select v-model="queryParams.level" filterable clearable :popper-append-to-body="false" placeholder="请选择">
                  <el-option v-for="(dict, index) in dict.type.high_light" :key="index" :label="dict.label" :value="dict.value">
                  </el-option>
                </el-select>
              </el-form-item>

              <el-form-item label="批次号" prop="lotNo">
                <el-input v-model="queryParams.lotNo" filterable clearable placeholder="请输入"></el-input>
              </el-form-item>

              <el-form-item label="管制项目名称" prop="spcName">
                <el-input v-model="queryParams.spcName" filterable clearable placeholder="请输入"></el-input>
              </el-form-item>

              <el-form-item label="触发类型" prop="triggerTypeList">
                <el-select v-model="queryParams.triggerTypeList" multiple clearable filterable collapse-tags :popper-append-to-body="false" placeholder="请选择">
                  <el-option v-for="(item, index) in dict.type.trigger_type" :key="index" :label="item.label"
                             :value="item.value"></el-option>
                </el-select>
              </el-form-item>

              <el-form-item label="多次异常" prop="isMultiple">
                <el-select v-model="queryParams.isMultiple" clearable filterable :popper-append-to-body="false" placeholder="请选择">
                  <el-option key="Y" label="Y" value="Y" />
                  <el-option key="N" label="N" value="N" />
                </el-select>
              </el-form-item>

              <el-form-item label="流程状态" prop="triggerTypeList">
                <el-select v-model="queryParams.triggerStatus"  clearable filterable collapse-tags :popper-append-to-body="false" placeholder="请选择">
                  <el-option v-for="(item, index) in dict.type.triger_status" :key="index" :label="item.label"
                             :value="item.value"></el-option>
                </el-select>
              </el-form-item>

              <el-form-item label="责任人" prop="owner">
                <el-input v-model="queryParams.owner" filterable clearable placeholder="请输入"></el-input>
              </el-form-item>
              <el-form-item label="测量项目" prop="testProjectName">
                <el-select
                  v-model="queryParams.testProjectName"
                  filterable
                  clearable
                >
                  <el-option
                    v-for="(item, index) in testProjectNameList"
                    :key="index"
                    :label="item"
                    :value="item"
                  ></el-option>
                </el-select>
              </el-form-item>
              <template v-for="field in this.dynamicList">
                <el-form-item :label="field.fieldNameDesc" :prop="field.fieldName" :key="field.fieldName">
                  <el-input v-if="field.htmlType === 'input'" v-model="queryParams[field.fieldName]" clearable ></el-input>
                  <el-select v-else-if="field.htmlType === 'select'" v-model="queryParams[field.fieldName]" multiple clearable filterable :popper-append-to-body="false" placeholder="请选择">
                    <el-option v-for="option in field.options" :key="option.dictValue" :label="option.dictLabel" :value="option.dictValue"></el-option>
                  </el-select>
                </el-form-item>
              </template>

              <el-form-item label="订单类型" prop="orderType">
                <el-select v-model="queryParams.orderType" filterable clearable :popper-append-to-body="false" placeholder="请选择">
                  <el-option v-for="(dict, index) in dict.type.order_type" :key="index" :label="dict.label" :value="dict.value">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </el-form-item>
    </el-form>
    <el-row :gutter="20" class="tool-bar">
      <el-col :span="12">
        <el-button type="warning" icon="el-icon-download" size="mini" plain @click="handleExport">导出</el-button>
        <el-button icon="el-icon-chat-line-square" size="mini" v-preventReClick="1000" @click="showRule">查看规则</el-button>
      </el-col>
      <el-col :span="12">
        <pagination
          v-if="queryParams.flag != 'total'"
          v-show="queryParams.total > 0"
          :total="queryParams.total"
          :page.sync="queryParams.pageNum"
          :limit.sync="queryParams.pageSize"
          @pagination="getList"
        /></el-col>
    </el-row>
    <Table ref="table" :queryParams="queryParams" :dynamic-list="dynamicList"></Table>
    <CommonDialog ref="commonDialog"/>
  </div>
</template>

<script>
import Table from "@/views/spc/ipp/ippTable";
import { download2 } from '@/utils/request';
import CommonDialog from "../components/CommonDialog";
import { factoryList } from "@/api/spc/spc";
import { getMesDeptList } from "@/api/mes/mesdata";
import {filterDictFactory} from "@/utils/ruoyi";
import { getDynamicByCpType } from '@/api/cp/extension'
import { getTestProjectNameListByCpType } from '@/api/cp/cpcondition'
import { getSpcGroupNameList } from '@/api/cp/ctrlplan'
export default {
  components: {
    Table,CommonDialog
  },
  name: "IPPList",
  computed:{


  },
  dicts: [
    'order_type', 'cp_param_level', 'cp_param_type', 'product_category',
    'trigger_type', 'high_light', 'pe_status', 'qe_status','triger_status'
  ],
  data() {
    return {
      searchShow: false,
      // dictsFactory:{},
      dynamicList:[],
      testProjectNameList:[],
      departmentList: [],
      factoryDepartmentList: [],
      dateRange: [],
      factoryList: [],
      queryParams: {
        total: 0,
        pageNum: 1,
        pageSize: 10,
        beginTime: '',
        endTime: '',
        orderType: '',
        category: '',
        variety: '',
        factory: '',
        factoryParams: [],
        dept: '',
        productFamily: '',
        pn: '',
        peStatus: '',
        qeStatus: '',
        triggerType: '',
        level: '',
        lotNo: '',
        spcName: '',
        owner: '',
        triggerTypeList: [],
        isMultiple: '',
        attribute6:"",attribute7:"",attribute8:"",attribute9:"",attribute10:"",
        attribute11:"",attribute12:"",attribute13:"",attribute14:"",attribute15:"",
        attribute16:"",attribute17:"",attribute18:"",attribute19:"",attribute20:"",
        testProjectName:""
      },
    };
  },
  created() {
    // this.initializationFactoryParams()
    this.factoryChange()
    /*    this.getFactoryList()
        this.getDepartmentList()*/
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
      if (this.queryParams.endTime && newValue) {
        this.queryParams.beginTime = new Date(newValue) > new Date(this.queryParams.endTime) ? oldValue : newValue;
      } else {
        this.queryParams.beginTime = newValue;
      }
    },
    'queryParams.endTime'(newValue, oldValue) {
      if (this.queryParams.beginTime && newValue) {
        this.queryParams.endTime = new Date(newValue) < new Date(this.queryParams.beginTime) ? oldValue : newValue;
      } else {
        this.queryParams.endTime = newValue
      }
    },
    /* 'queryParams.factoryParams'(newValue, oldValue){
       this.factoryDepartmentList = this.departmentList.filter(o => {
         return newValue.indexOf(o.factory) > -1
       })
       this.updateDeptValue(newValue)
     }*/
  },
  methods: {
    async getDynamicByCp(){
      getDynamicByCpType({factory:this.queryParams.factoryParams[0],cpType:"IPP"}).then(response=>{
        this.dynamicList = response.data
      })
    },
    initializationFactoryParams(){
      this.queryParams.factoryParams = [this.$store.state.user.currAppFactoryList[0].code];
    },
    /*getDepartmentList() {
      getMesDeptList().then(response => {
        this.departmentList = response.data
      })
    },
    getFactoryList() {
      factoryList().then(response => {
        this.factoryList = response.data
      })
    },*/

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
        "/chinafastprint/ippInfo/exportIpp",
        {
          ...copyQueryParams,
        },
        `IPP_${new Date().getTime()}.xlsx`
      );
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.dynamicList=[];
      this.dateRange = [];
      this.resetForm("queryForm");
      this.handleQuery();
      this.factoryChange();
    },
    showRule() {
      this.$refs.commonDialog.show()
    },

    // 日期范围选择
    // queryDateRange(dateRange){
    //   this.queryParams.beginTime = dateRange[0] || '';
    //   this.queryParams.endTime = dateRange[1] || '';
    // },
    getList() {
      this.$refs.table.getList();
    },
    factoryChange(){
      this.queryParams.testProjectName=''
      this.getTestProjectNameList()
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
    getTestProjectNameList(){
      getTestProjectNameListByCpType({
        factoryParams: this.queryParams.factoryParams,
        cpType: "IPP",
      }).then(response => {
        const dataList = response.data;
        this.testProjectNameList = dataList;
      });
    }
    /*    updateDeptValue(newValue){
          if(this.queryParams.dept) {
            let selectDept = this.departmentList.filter(t => t.factoryJointId === this.queryParams.dept)
            if (selectDept) {
              if(newValue.indexOf(selectDept[0].factory)<=-1){
                this.queryParams.dept=''
              }
            }else{
              this.queryParams.dept=''
            }
          }
        }*/
  },
};
</script>
<style>
.tool-bar {
  margin: 15px 0;
  display: flex;
  justify-content: space-between;
  .pagination-container {
    position: relative;
    height: 30px;
    padding: 0 !important;
    margin: 0 !important;
  }
}
/**/
/* .trend canvas{
  height: 300px!important;
  left: -30px!important;
} */
</style>
<!--
<style scoped>
.app-container {
  padding: 6px 20px 20px 20px;
}
>>> .queryCondition .el-form-item__label {
  width: 68px;
  padding: 0 2px 0 0;
}
>>> .queryCondition .el-form-item--medium * {
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
</style>

-->

