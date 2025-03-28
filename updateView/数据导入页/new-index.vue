<template>
  <div class="app-container">
    <div>
      <el-dialog :visible.sync="dialogVisible" width="35%"  append-to-body class="scrollbar" title="错误提示">
        <div>
          <ul class="dialog_ul">
            <li v-for="(content, index) in msg" :key="index" class="dialog_li">
              <span>{{ content }}</span>
            </li>
          </ul>
        </div>
      </el-dialog>
    </div>
    <el-row>
      <select-factory v-model="factory" size="mini" filterable  placeholder="请选择数据导入所属厂别" @change="factoryChange"></select-factory>
      <el-button type="primary" size="mini" @click="addRow" :disabled="this.factory===''">新增行</el-button>
      <el-button type="primary" size="mini" @click="handleImport" :disabled="this.factory===''">导入</el-button>
      <el-button type="primary" size="mini" @click="handleExport">导出</el-button>
      <el-button type="success" size="mini" @click="checkInfo" >校验</el-button>
      <el-button type="primary" size="mini" @click="updateInfo" >修改</el-button>
      <el-button type="primary" size="mini" @click="saveInfo" :loading="isLoading"  :disabled="!isSave" v-show="!isProceed">保存</el-button>
      <el-button type="primary" size="mini" @click="cleanTable">清空</el-button>
      <el-button type="success" size="mini" @click="saveInfo" v-show="isProceed" >继续入库</el-button>

      <el-tag type="warning" style="margin-left: 10px;">
        <i class="el-icon-warning"></i>
        <span>温馨提示：</span>
        <strong>先校验再保存</strong>
      </el-tag>




    </el-row>
    <u-table
      v-if="loadHeader"
      class="logTable"
      :data="tableData"
      :data-changes-scroll-top="false"
      use-virtual
      :row-height="45"
      :height="680"
      :row-class-name="tableRowClassName"
      style="width: 100%">
      <u-table-column type="index" label="#" width="60" />
      <u-table-column prop="factory" label="厂别" width="100">
        <template slot-scope="scope">
          <el-select size="mini" v-model="scope.row.factory" clearable :disabled="true">
            <el-option v-for="item in factoryList" :key="item.factoryName" :label="item.label" :value="item.factoryName"></el-option>
          </el-select>
        </template>
      </u-table-column>
      <u-table-column prop="analysisType" label="药水类型" width="100">
        <template slot-scope="scope">
          <el-select size="mini" v-model="scope.row.analysisType" clearable :disabled="isCheck">
            <el-option v-for="item in dict.type.chemical_type" :key="item.value" :label="item.value" :value="item.value"></el-option>
          </el-select>
        </template>
      </u-table-column>
      <u-table-column prop="measureMachine" label="设备编号" width="100">
        <template slot-scope="scope">
          <el-input size="mini" v-model="scope.row.measureMachine" :disabled="isCheck"/>
        </template>
      </u-table-column>
      <u-table-column prop="sampleReceivingTime" label="送样时间" width="180">
        <template slot-scope="scope">
          <el-date-picker
            size="mini"
            style="width: 100%"
            v-model="scope.row.sampleReceivingTime"
            type="datetime"
            format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"
            placeholder="选择日期" :disabled="isCheck"/>
        </template>
      </u-table-column>
      <u-table-column prop="testTime" label="测量时间" width="180">
        <template slot-scope="scope">
          <el-date-picker
            size="mini"
            style="width: 100%"
            v-model="scope.row.testTime"
            type="datetime"
            format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"
            placeholder="选择日期" :disabled="isCheck"/>
        </template>
      </u-table-column>
      <!-- <u-table-column prop="triggerDeliveryTime" label="生成检验单时间" width="100">
          <template slot-scope="scope">
              <el-date-picker
                  v-model="scope.row.triggerDeliveryTime"
                  type="datetime"
                  format="yyyy-MM-dd HH:mm:ss" value-format="yyyy-MM-dd HH:mm:ss"
                  placeholder="选择日期" />
          </template>
      </u-table-column> -->
      <u-table-column prop="testValue" label="测量值" width="100">
        <template slot-scope="scope">
          <el-input size="mini" v-model="scope.row.testValue" :disabled="isCheck"/>
        </template>
      </u-table-column>
      <!-- <u-table-column prop="spcType" label="类型" width="100">
          <template slot-scope="scope">
              <el-input v-model="scope.row.spcType" disabled />
          </template>
      </u-table-column> -->
      <u-table-column prop="spcGroup" label="群组" width="100">
        <template slot-scope="scope">
          <el-input size="mini" v-model="scope.row.spcGroup" :disabled="isCheck"/>
        </template>
      </u-table-column>
      <u-table-column prop="spcFileName" label="档案" width="100">
        <template slot-scope="scope">
          <el-input size="mini" v-model="scope.row.spcFileName" :disabled="isCheck"/>
        </template>
      </u-table-column>
      <u-table-column prop="spcName" label="管制项目名称" width="100">
        <template slot-scope="scope">
          <el-input size="mini" v-model="scope.row.spcName" :disabled="isCheck"/>
        </template>
      </u-table-column>

      <u-table-column prop="createBy"
                      label="账号名"
                      width="80">
      </u-table-column>
      <u-table-column prop="createTime"
                      label="导入时间"
                      width="100">
      </u-table-column>
      <template v-for="field in this.dynamicList">
        <u-table-column :label="field.fieldNameDesc" :prop="field.fieldName" width="100">
          <template slot-scope="scope">
            <el-input size="mini" v-if="field.htmlType === 'input'" v-model="scope.row[field.fieldName]" clearable ></el-input>
            <el-select size="mini" v-else-if="field.htmlType === 'select'" v-model="scope.row[field.fieldName]"  clearable filterable  placeholder="请选择">
              <el-option v-for="option in field.options" :key="option.dictValue" :label="option.dictLabel" :value="option.dictValue"></el-option>
            </el-select>
          </template>
        </u-table-column>
      </template>
      <u-table-column prop="operate"
                      label="操作"
                      width="80">
        <template slot-scope="scope">
          <a @click="delRow(scope.$index)">删除</a>
        </template>
      </u-table-column>

    </u-table>
    <el-dialog :title="upload.title" :visible.sync="upload.open" width="400px" append-to-body>
      <el-upload
        ref="upload"
        :limit="1"
        accept=".xlsx, .xls"
        :headers="upload.headers"
        :action="upload.url"
        :disabled="upload.isUploading"
        :on-progress="handleFileUploadProgress"
        :on-success="handleFileSuccess"
        :auto-upload="false"
        :data="{'key':'chemical','factory':this.factory}"
        drag>
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
        <div class="el-upload__tip text-center" slot="tip">
          <span>仅允许导入xls、xlsx格式文件。</span>
          <el-link type="primary" :underline="false" style="font-size:12px;vertical-align: baseline;" @click="importTemplate">下载模板</el-link>
        </div>
      </el-upload>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitFileForm">确 定</el-button>
        <el-button @click="upload.open = false">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import { getToken } from "@/utils/auth";
import { download2 } from '@/utils/request';
import { getMesDeptList } from "@/api/mes/mesdata";
import {factoryList, chemicalSaveData, getBatchId, chemicalCheckData} from "@/api/spc/spc";
import { getDynamicByCpType } from '@/api/cp/extension';
// import { json } from "stream/consumers";
export default {

  components:{

  },
  name: 'ChemicalImport',
  dicts: ['chemical_type', 'Area','Point','ClearLevel','triger_status','high_light','floor','particle_diameter'],
  data() {
    return {
      //动态表头重加载
      loadHeader:false,
      factory:"",
      dynamicList:[],
      msg: [],
      factoryList: [],
      departmentList: [],
      tableData: [],
      saveData: [],
      isCheck: false,
      isSave: false,
      isProceed: false,
      exportUrl: "/spc/chemicalExportData",
      isLoading: false,
      isCheckLoading: false,

      dialogVisible: false,
      autoCloseTimer: null,
      flagList: [],
      // Excel导入参数
      upload: {
        // 是否显示弹出层
        open: false,
        // 弹出层标题
        title: "Chemical导入",
        // 是否禁用上传
        isUploading: false,
        // 是否更新已经存在的用户数据
        updateSupport: 0,
        // 设置上传的请求头部
        headers: {appCode: this.$store.state.user.code,Authorization: "Bearer " + getToken() },
        // 上传的地址
        url: process.env.VUE_APP_BASE_API + "/spc/importData"
      },
      data: {
        buildingFloor:"1",
        departmentProcessModule: "1",
        measureArea: "",
        measurePoint: "",
        cleanLinessLevel: "3",
        particleDiameter: "",
        testValue: "",
        testTime: "",
        operate: "",
        createTime: '',
        createBy: ''
      },
    }
  },
  created(){
    this.getDepartmentList()
    this.getFactoryList()
  },
  mounted() {

  },
  methods: {
    async getDynamicByCp(){
      this.loadHeader = false;
      getDynamicByCpType({factory:this.factory,cpType:"CHEMICAL"}).then(response=>{
        this.dynamicList = response.data;
        this.loadHeader = true;
      })
    },
    factoryChange(){
      this.getDynamicByCp();
      //如果已经上传了数据后再次修改工厂,修改数据中的工厂字段
      if(this.tableData.length>0){
        this.tableData.forEach(item=>{
          item.factory =this.factory;
        })
      }
    },
    tableRowClassName({row, rowIndex}) {
      let index = rowIndex+1;
      if (this.flagList.includes(index)){
        return 'change-row';
      }else {
        return '';
      }
    },
    startAutoClose() {
      if (this.autoCloseTimer) clearTimeout(this.autoCloseTimer);
      this.autoCloseTimer = setTimeout(() => {
        this.dialogVisible = false;
      }, 10000);
    },
    getFactoryList(){
      factoryList().then(response=>{
        this.factoryList = response.data
      })
    },
    getDepartmentList(){
      getMesDeptList().then(response=>{
        this.departmentList = response.data
      })
    },
    addRow() {
      this.msg = [];
      this.isProceed = false;
      this.isSave = false;
      this.isCheck = false;
      let obj = {
        factory: this.factory,
        analysisType: "",
        measureMachine: "",
        sampleReceivingTime: "",
        testTime: "",
        triggerDeliveryTime: "",
        testValue: "",
        spcType: "CHEMICAL",
        spcGroup: "",
        spcFileName: "",
        spcName: "",
      };
      this.tableData.push(obj);

    },
    delRow(index) {
      this.isSave = false;
      this.tableData.splice(index, 1)
    },
    cleanTable() {
      this.flagList = [];
      this.msg = [];
      this.$confirm('是否清空所录入的内容?', '提示', {
        confirmButtonText: '是',
        cancelButtonText: '否',
        type: 'warning'
      }).then(() => {
        this.isProceed = false;
        this.isSave = false;
        this.isCheck = false;
        this.tableData = [];
        this.$message({
          type: 'success',
          message: '删除成功!'
        });
      }).catch(() => {

      });
    },
    updateInfo() {
      this.msg = [];
      this.isProceed = false;
      this.isSave = false;
      this.isCheck = false;
    },

    checkInfo() {
      this.isCheckLoading = true;
      this.msg = [];
      this.isProceed = false;
      if(!this.tableData || this.tableData.length == 0) {
        this.$message({
          type: 'error',
          message: '数据为空'
        });
        return;
      }
      /* this.isLoading = true;*/
      getBatchId(this.tableData.length).then(res => {
        let msgArr = [];
        let inx = 0;
        let checkList = [];
        let indexStr = '';
        let indexDataStr='';
        for(let info of this.tableData) {
          let check = info['spcGroup'] + info['spcFileName'] + info['spcName'] + info['testTime'];
          if(checkList.indexOf(check) > -1) {
            msgArr.push("第"+(checkList.indexOf(check)+1)+"行与第"+(inx + 1)+"行测试时间相同，请检查")

            indexStr += ',' + (inx + 1);
          }
          checkList.push(check);
          info['batchId'] = res[inx];
          if(!(info['spcGroup']&&info['spcFileName'] && info['spcName'] && info['testTime'])){
            indexDataStr += ',' + (inx + 1);
          }
          inx++;
        }
        if(indexStr) {

          if(indexStr){
            let msg = msgArr.join('<br/>')
            this.$message({
              dangerouslyUseHTMLString: true,
              message: msg,
              type: 'error'
            })
            return;
          }
          /* this.isLoading = false;*/
          this.$message({
            type: 'error',
            message: (indexStr?'请检查以下行号:'+ indexStr.substring(1,indexStr.length)+';':'')+(indexDataStr?'数据有误行号:'+indexDataStr.substring(1,indexDataStr.length):''),
          });
          return;
        }
        this.tableData.forEach(function(element, index) {

          element.index = index+1;
          element.spcType = 'CHEMICAL';
        });
        this.$modal.loading("校验中，请稍候...");
        chemicalCheckData(this.tableData).then(res => {
          this.flagList = [];
          this.isCheckLoading = false;
          if(res.code=='200'){
            this.isCheck = true;

            this.saveData = res.data.infos;


            if(res.data.isProceed === '1'){
              this.isProceed = true;
              let msgArr = [];
              msgArr = res.data.errors;
              let msg = msgArr.join('<br/>')
              this.msg = msgArr;
              this.dialogVisible = true;

              this.flagList = res.data.flagList;
              this.startAutoClose();
            }else {
              this.isSave = true;
              this.$message({
                type: 'success',
                message: '校验成功!'
              });

            }
          }else{
            this.$message({
              type: 'error',
              message: res.msg
            });
          }

        }).catch(err => {
          this.isCheckLoading = false;
          this.$message({
            type: 'error',
            message: '校验失败'
          });
        }).finally(() =>{
          this.$modal.closeLoading();
        });
      }).catch(err => {
        this.isCheckLoading = false;
        this.isCheckLoading = false;
        this.$message({
          type: 'error',
          message: '获取批次号失败'
        });
      });
    },

    saveInfo() {
      this.msg = [];
      this.isProceed = false;
      if(!this.saveData || this.saveData.length == 0) {
        this.$message({
          type: 'error',
          message: '数据为空'
        });
        return;
      }
      this.isLoading = true;
      chemicalSaveData(this.saveData).then(res => {
        this.isLoading = false;
        if(res.code=='200'){
          this.isSave = false;
          for(let val of this.tableData) {
            this.$set(val, 'createBy', res.data.userName);
            this.$set(val, 'createTime', res.data.saveTime);
          }
          this.$message({
            type: 'success',
            message: '保存成功!'
          });
        }else{
          this.$message({
            type: 'error',
            message: res.msg
          });
        }

      }).catch(err => {
        this.isSave = false;
        this.isProceed = false;
        this.isLoading = false;
        this.$message({
          type: 'error',
          message: '保存失败'
        });
      });
    },
    // 提交上传文件
    submitFileForm() {
      this.$refs.upload.submit();
    },
    // 文件上传中处理
    handleFileUploadProgress(event, file, fileList) {
      this.upload.isUploading = true;
    },
    // 文件上传成功处理
    handleFileSuccess(response, file, fileList) {
      this.upload.open = false;
      this.upload.isUploading = false;
      this.$refs.upload.clearFiles();
      this.$alert("<div style='overflow: auto;overflow-x: hidden;max-height: 70vh;padding: 10px 20px 0;'>" + response.msg + "</div>", "导入结果", { dangerouslyUseHTMLString: true });
      response.data.forEach(item => {
        if(item.testTime) {
          item.testTime = new Date(item.testTime.replace('CST', 'GMT+0800')).format("yyyy-MM-dd hh:mm:ss");
        }
        if(item.triggerDeliveryTime) {
          item.triggerDeliveryTime = new Date(item.triggerDeliveryTime.replace('CST', 'GMT+0800')).format("yyyy-MM-dd hh:mm:ss");
        }
        if(item.sampleReceivingTime) {
          item.sampleReceivingTime = new Date(item.sampleReceivingTime.replace('CST', 'GMT+0800')).format("yyyy-MM-dd hh:mm:ss");
        }
        if(!item.spcType) {
          item.spcType = 'CHEMICAL'
        }
        this.tableData.push(item);
      });
    },
    /** 导出模板 */
    importTemplate() {
      download2(this.exportUrl, [] , `chemical_template_${new Date().getTime()}.xlsx`)
    },
    /** 导入按钮操作 */
    handleImport() {
      this.isProceed = false;
      this.isSave = false;
      this.isCheck = false;
      this.upload.open = true;
    },
    /** 导出按钮操作 */
    handleExport() {
      download2(this.exportUrl, this.tableData , `chemical_${new Date().getTime()}.xlsx`)
    },
  }
};
</script>
<style scoped>
>>> .u-table th.u-table__cell > .cell{
  padding: 0px;
}
.u-table .cell {
  padding: 0px;
}
</style>
<style scoped>
.dialog_ul {
  padding: 0;
  margin: 0;
  list-style-type: none
}

.dialog_li {
  margin-bottom: 5px;
  font-size: 14px;
  color: #e6a23c;
}

strong {
  margin-bottom: 5px;
  font-size: 12px;
  color: #e6a23c;
}

.logTable {
  margin-top: 10px;
}

.logTable /deep/ .change-row,.logTable /deep/ .change-row input, .logTable /deep/ select{
  color: #e6a23c;
}

</style>
