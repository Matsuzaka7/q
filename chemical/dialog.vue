<template>
  <div>
    <el-dialog class="el-dialog-cus" v-bind="attributes" :visible="visible" :before-close="beClose" append-to-body
    :close-on-click-modal="false" :modal-append-to-body="false" @opened="loadData()" v-on="on" width="90%">
    <!-- @opened="loadData()" -->
    <span>{{this.title}}</span>
    <slot v-if="visibleSlot"></slot>

    <el-tabs v-model="activeName" @tab-click="tabClick">
      <el-tab-pane label="OCAP" name="ocap" v-if="ocap">
        <ocap-bill ref="ocapBill" />
      </el-tab-pane>
      <el-tab-pane label="监控图" name="monitor" v-if="monitor">
        <chemicalMonitor :id="id" :num="num+1" ref="chemicalMonitor" />
      </el-tab-pane>
      <el-tab-pane label="ME/QE处置" name="peqe" v-if="peqe">
        <el-form :model="form" ref="peqeForm" :rules="meQeRule" label-width="90px">
          <div class="module-line">
            <h3 class="module-title">ME原因说明</h3>
          </div>
          <el-form-item label="原因分析" prop="peReason">
            <el-input :disabled="!enableME || num > 1" type="textarea" :rows="4" v-model="form.peReason"></el-input>
          </el-form-item>
          <el-form-item label="任务" prop="taskList">
            <el-table :data="taskList" border>
              <el-table-column label="序号" type="index" width="55" align="center" />
              <el-table-column label="措施内容" prop="actionName" align="center" />
              <el-table-column label="CA/PA" prop="actionType" align="center" />
              <el-table-column label="部门" prop="causedDept" align="center" />
              <el-table-column label="责任人" prop="owner" align="center" />
              <el-table-column label="截止日期" prop="ecd" align="center" />
              <el-table-column label="状态" prop="status" align="center" />
              <el-table-column label="执行情况说明" prop="executeExplain" align="center" />
              <el-table-column label="附件" prop="accessory" align="center">
                <template slot-scope="scope">
                  <fileUpload v-model="scope.row.accessory" :readonly="true" />
                </template>
              </el-table-column>
              <el-table-column label="Open项完成者" prop="finisher" align="center">
                <template slot-scope="scope">
                  {{ scope.row.finisher ? scope.row.finisher :'/' }}
                </template>
              </el-table-column>

              <el-table-column label="Open项完成时间" prop="finisherTime" align="center">
                <template slot-scope="scope">
                  {{ scope.row.finisherTime ?scope.row.finisherTime: '/' }}
                </template>
              </el-table-column>

              <el-table-column label="Open项确认者" prop="confirmer" align="center">
                <template slot-scope="scope">
                  {{ scope.row.confirmer ? scope.row.confirmer: '/' }}
                </template>
              </el-table-column>
              <el-table-column label="驳回原因" prop="rejectReason" align="center" />
              <el-table-column label="操作" align="center" width="80" v-if="(enableME || openAction) && !(num > 1)">
                <template slot-scope="scope">
                  <el-form-item v-if="scope.row.status === 'open' || scope.row.status === 'forward'">
                    <el-button
                        size="mini"
                        type="text"
                        icon="el-icon-edit"
                        @click="taskUpdate(scope.row)"
                    >编辑</el-button>
                  </el-form-item>
                  <el-form-item v-if="!openAction && permission.meRemove">
                    <el-button
                        size="mini"
                        type="text"
                        icon="el-icon-delete"
                        @click="taskDelete(scope.row)"
                    >删除</el-button>
                  </el-form-item>
                  <template v-if="(scope.row.status !== 'open' && !(!openAction && permission.meRemove))">/</template>
                </template>
              </el-table-column>
            </el-table>
          </el-form-item>
          <el-form-item v-if="enableME">
            <div class="left" v-if="permission.meAdd && !(num > 1)">
              <el-button type="primary" @click="taskAdd">新增</el-button>
            </div>
          </el-form-item>
          <el-form-item label="" v-if="form.isMultiple == 'Y'">
            前次发生的link
            <el-button type="primary" @click="$refs.PointDialog.id=form.lastId;$refs.PointDialog.open(()=>{})" :disabled="(num > 2)">前次异常</el-button>
          </el-form-item>
          <el-form-item label=" " prop="checkTriggers" style="margin-bottom: 0px" v-if="form.isMultiple == 'Y'">
            <el-table :data="form.checkTriggers" border >
              <el-table-column label="" type="index" prop="num" width="55" align="center" />
              <el-table-column label="多点OOC/OOS触发确认checkList" prop="checkMsg" align="center" />
              <el-table-column label="确认结果" align="center">
                <template slot-scope="socpe">
                  <!-- {{scope.row.checkResult}} -->
                  <el-input v-model="socpe.row.checkResult" maxlength="500" :disabled="!enableME || (num > 1)"></el-input>
                </template>
              </el-table-column>
              <el-table-column label="确认人" prop="checkPeople" align="center" />
              <el-table-column label="确认时间" prop="checkTime" align="center" />
            </el-table>
          </el-form-item>
          <el-form-item label=" " prop="checkAccessory" v-if="form.isMultiple == 'Y'">
            <div style="border: 1px solid #dfe6ec">
              附件
              <fileUpload v-model="form.checkAccessory" :readonly="!enableME || (num > 1)" />
            </div>
          </el-form-item>
          <el-form-item v-if="enableME && !(num > 1)">
            <div class="right">
              <el-button type="primary" @click="dialogButton('save', 'me')" v-if="permission.meSave">保存</el-button>
              <el-button type="primary" @click="dialogButton('commit', 'me')" v-if="permission.meCommit">提交</el-button>
            </div>
          </el-form-item>
          <div class="module-line">
            <h3 class="module-title">QE确认</h3>
          </div>
          <el-form-item label="风险评估" prop="qeRisk">
            <el-input :disabled="!enableQE" type="textarea" :rows="1" style="width:50%;" v-model="form.qeRisk"></el-input>
          </el-form-item>
          <el-form-item label="CA/PA措施有效性评估" prop="qeValidity" class="long-label">
            <el-input :disabled="!enableQE" type="textarea" :rows="6" style="width:50%;"
              v-model="form.qeValidity"></el-input>
          </el-form-item>
          <el-form-item label="附件" prop="accessory">
            <fileUpload v-model="form.accessory" :readonly="!enableQE || !permission.qeUpload" />
          </el-form-item>
          <el-form-item align="right" v-if="form.id != '' && enableQE">
            <el-button type="primary" @click="dialogButton('save', 'qe')" v-if="permission.qeSave">保存</el-button>
            <el-button type="primary" @click="dialogButton('commit', 'qe')" v-if="permission.qeCommit">提交</el-button>
            <el-button type="primary" @click="$refs.rejectForm.openDialog(form)" v-if="form.peStatus == 'forward' && permission.qeBack">驳回</el-button>
            <el-button type="primary" @click="dialogButton('close', 'qe')" v-if="permission.qeClose">结案</el-button>
          </el-form-item>
        </el-form>
        <rejectForm ref="rejectForm" @ok="backForm" />
        <chemical-dialog ref="PointDialog" :num="(num + 1)"/>
      </el-tab-pane>
      <el-tab-pane label="历史记录" name="history" v-if="history">

        <el-table :data="loglList" style="width: 100%;">
          <el-table-column type="index" label="序号" width="55">
          </el-table-column>
          <el-table-column prop="createTime" label="时间" width="180">
          </el-table-column>
          <el-table-column prop="createBy" label="操作人" width="180">
          </el-table-column>
          <el-table-column prop="msg" label="日志描述" width="180">
          </el-table-column>
          <el-table-column prop="rejectReason" label="驳回原因" width="180">
          </el-table-column>
          <el-table-column prop="changeReason" label="变更原因" width="180">
          </el-table-column>
        </el-table>

      </el-tab-pane>
    </el-tabs>
    <!--  Task表单  -->
    <el-dialog :title="taskTitle" :visible.sync="taskOpen" :close-on-click-modal="false" width="800px" append-to-body>
      <el-form ref="taskForm" :model="taskForm" :rules="taskRules" label-width="120px">
<!--        <el-form-item label="厂别" prop="factory">
          <el-select v-model="taskForm.factory" filterable clearable placeholder="请选择" :disabled="openAction" >
            <el-option v-for="item in factoryList"  :key="item.factoryName" :label="item.label" :value="item.factoryName">
            </el-option>
          </el-select>
        </el-form-item>-->

        <el-form-item label="厂别" prop="factory">
          <el-select v-model="taskForm.factory" filterable clearable @change="onFactoryChange">
            <el-option
              v-for="item in factoryList"
              :key="item.factoryName"
              :label="item.factoryName"
              :value="item.factoryName"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="措施内容" prop="actionName">
          <el-input v-model="taskForm.actionName" :disabled="openAction" type="textarea" placeholder="请输入措施内容" />
        </el-form-item>
        <el-form-item label="CA (纠正措施) /PA (预防措施)" prop="actionType">
          <el-select v-model="taskForm.actionType" :disabled="openAction">
            <el-option value="CA"></el-option>
            <el-option value="PA"></el-option>
          </el-select>
        </el-form-item>
<!--        <el-form-item label="部门" prop="causedDept">
          <el-select v-model="taskForm.causedDept" :disabled="openAction" @change="getMesEmployeeList">
            <el-option v-for="(dict,index) in departmentList" :key="index" :label="dict.factory+'-'+dict.deptName" :value="dict.deptName">
            </el-option>
          </el-select>
        </el-form-item>-->

        <el-form-item label="部门" prop="causedDept">
          <el-select
            v-model="taskForm.causedDept"
            filterable
            clearable
            collapse-tags
            placeholder="请选择"
            @change="onDeptChangeNew"
          >
            <el-option
              v-for="(item, index) in departmentList"
              :key="item.deptName"
              :label="item.deptName"
              :value="item.deptName"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="责任人" prop="owner">
          <el-select v-model="taskForm.owner"
                     filterable
                     clearable
                     collapse-tags>
            <el-option v-for="(d,index) in userList" :key="index" :label="d.factory+'-'+d.userName" :value="d.userName">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="截止日期" prop="ecd">
          <el-date-picker v-model="taskForm.ecd" :disabled="openAction" type="datetime" value-format="yyyy-MM-dd HH:mm:ss" default-time="23:59:59"
            placeholder="选择日期"></el-date-picker>
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-select v-model="taskForm.status" :disabled="openAction">
            <el-option value="open"></el-option>
            <el-option value="close"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="执行情况说明" prop="executeExplain">
          <el-input v-model="taskForm.executeExplain" type="textarea" placeholder="请输入执行情况说明" />
        </el-form-item>
        <el-form-item label="附件上传" prop="accessory">
          <fileUpload v-model="taskForm.accessory" />
        </el-form-item>
      </el-form>
      <rejectForm ref="rejectTaskForm" @ok="backTaskForm" />
      <div slot="footer" class="dialog-footer">
        <template v-if="openAction">
          <el-button type="primary" @click="submitTaskForm('commit')">提 交</el-button>
          <template v-if="enableTask">
            <el-button type="primary" @click="$refs.rejectTaskForm.openDialog(taskForm)">驳 回</el-button>
            <el-button type="primary" @click="submitTaskForm('close')">结 案</el-button>
          </template>
        </template>
        <el-button type="primary" @click="submitTaskForm('save')">保 存</el-button>
        <el-button @click="taskCancel">取 消</el-button>
      </div>
    </el-dialog>
  </el-dialog>
  </div>
</template>

<script>
import {addTask, editTask, getTask, getTaskList, removeTask, factoryList} from "@/api/spc/spc";
import FmTab from "@/views/spc/fm/monitor";
import chemicalMonitor from "@/views/spc/chemical/monitor";
import {applyInfo, getInfo, postInfo} from "@/api/qdm/chemical";
import fileUpload from '@/views/spc/chemical/fileUpload';
import {listLog} from '@/api/qdm/log'
import {getMesDeptList, getMesDeptListByFactoryNew, getMesEmployeeListByName} from "@/api/mes/mesdata";
import ocapBill from '@/views/ocap/bill/index';
import rejectForm from '../components/rejectForm.vue';

export default {
  name: 'ChemicalDialog',
  components: {rejectForm, FmTab, chemicalMonitor, fileUpload, ocapBill },
  // inheritAttrs: false,
  props: {
    num: {
      type: Number,
      default: -1,
    },
    autoClose: {
      type: Boolean,
      default: true,
    },
    beforeClose: {
      type: Function,
      default: () => { },
    },
    id: String,
    monitorTable: {      //监控图，折线图数据
      Object,
      default: () => { }
    },
    historyTable: {     //history页签数据
      Array,
      default: () => [
        { id: '1', operator: 'sunny', time: '2022/12/12 12:00:00', described: '触发Area' },
        { id: '2', operator: 'sunny', time: '2022/12/12 12:00:00', described: '保存ME信息' },
        { id: '3', operator: 'sunny', time: '2022/12/12 12:00:00', described: '提交' }

      ]
    },
    ocap: {           //是否展示ocap页面
      Boolean,
      default: true
    },
    monitor: {        //是否展示监控图
      Boolean,
      default: true
    },
    peqe: {
      Boolean,
      default: true
    },
    history: {
      Boolean,
      default: true
    }
  },
  dicts: [
    'ipp_department'
  ],
  data() {
    var checkAccessoryV = (rule, value, callback) => {
       if (value==null || value=='[]' || value.length==0){
        callback(new Error('请添加附件'));
      }else {
        callback();
      }
    };
    var accessoryV = (rule, value, callback) => {
      if (this.taskForm.status==='close' && (this.taskForm.accessory==null || this.taskForm.accessory.length<3)) {
        callback(new Error('close状态需要上传附件，请添加'));
      }
       else if (this.openAction && (this.taskForm.accessory==null || this.taskForm.accessory.length<3)){
        callback(new Error('请添加附件'));
      }else {
        callback();
      }
    };
    const { top = '20vh', width = '420px', title = 'SPC Chemical Trigger Detail', center = false, btnTxt = ["取消", "确定"] } = this.config || {};
    const validateTaskList = (rule, value, callback) => {
      if (this.taskList == 0) {
        callback(new Error("请添加任务"));
      } else {
        callback();
      }
    };
    const validateCheckList = (rule, value, callback) => {
      let msg = "";
      for(let obj of value) {
        if(!obj.checkResult) { msg="请填写checkList确认结果"; }
      }
      if (msg) {
        callback(new Error(msg))
      } else {
        callback();
      }
    };
    return {
      title: '',
      config: {
        top: "20vh",
        width: "500px",
        center: true,
        btnTxt: ["取消", "提交"],
      },
      lastId: undefined,
      departmentList: [],
      visible: false,
      activeName: "peqe",
      factoryList: [],
      attributes: {
        top,
        width,
        title,
        center,
        ...this.config,
      },
      meQeRule: {
        peReason: [{
          required: true, message: "请输入", trigger: "blur"
        }],
        taskList: [{
          required: true, validator: validateTaskList, trigger: "change"
        }],
        checkTriggers: [{
          required: true, validator: validateCheckList, trigger: "change"
        }],
        qeRisk: [
          { required: true, message: "请输入", trigger: "blur" }
        ],
        qeValidity: [
          { required: true, message: "请输入", trigger: "blur" }
        ],
        checkResult: [
          {required: true, message: "请输入", trigger: "blur"}
        ],
        checkAccessory: [
          { required: true, validator:checkAccessoryV,  trigger: "change" }
        ],
      },
      loglList: [],
      btnTxt,
      on: this.getDialogEvents(),
      visibleSlot: false,
      form: {},
      taskList: [],
      checkTriggers:[],
      taskForm: {
        status:"",
        accessory:""
      },
      taskTitle: '编辑Task',
      taskOpen: false,
      taskRules: {
        actionName: [
          { required: true, message: "请输入", trigger: "blur" }
        ],
        actionType: [
          { required: true, message: "请选择", trigger: "change" }
        ],
        causedDept: [
          { required: true, message: "请选择", trigger: "blur" }
        ],
        owner:[
          { required: true, message: "请选择", trigger: "blur" }
        ],
        ecd:[
          { required: true, message: "请选择", trigger: "change" }
        ],
        status:[
          { required: true, message: "请选择", trigger: "change" }
        ],
        executeExplain: [
          { required: true, message: "请输入", trigger: "blur" }
        ],
        factory: [
          { required: true, message: "请输入", trigger: "blur" }
        ],
        accessory: [
          { validator:accessoryV,  trigger: "change" }
        ],
      },
      userList: [],
      permission: {}
    };
  },
  computed: {
    enableME() {
      return this.form.peStatus == 'open';
    },
    enableQE() {
      return this.form.peStatus == 'forward' && this.form.triggerStatus != 'close' && this.form.qeStatus != 'close';
    },
    openAction() {
      return this.form.triggerStatus == 'close';
    },
    enableTask() {
      return this.$store.state.user.id == this.taskForm.requestor;
    }
  },
  created() {
    /*this.getDepartmentList();*/
    this.initPermission();
    this.getFactoryList();
  },
  methods: {
    initPermission() {
      const permissions = {
        meAdd: 'chinafastprint:me:add',
        meEdit: 'chinafastprint:me:edit',
        meRemove: 'chinafastprint:me:remove',
        meSave: 'chinafastprint:me:save',
        meCommit: 'chinafastprint:me:commit',
        qeSave: 'chinafastprint:qe:save',
        qeCommit: 'chinafastprint:qe:commit',
        qeBack: 'chinafastprint:qe:back',
        qeClose: 'chinafastprint:qe:close',
        qeUpload: 'chinafastprint:qe:upload'
      };
      for (const k in permissions) {
        this.permission[k] = this.$auth.hasPermi(permissions[k]);
      }
    },
    /*getDepartmentList() {
      getMesDeptList().then(response => {
        debugger
        this.departmentList = response.data
      })
    },*/
    getDepartmentList() {
      getMesDeptListByFactoryNew({factory:this.taskForm.factory}).then((response) => {
        this.departmentList = response.data;
      });
    },
    getFactoryList(){
      factoryList().then(response=>{
        this.factoryList = response.data
      })
    },
    // 获取责任人
    getMesEmployeeList() {
      if (this.taskForm.causedDept) {
        getMesEmployeeListByName({deptName: this.taskForm.causedDept,factory: this.taskForm.factory}).then(response => {

          this.userList = response.data;
        });
      } else {
        this.userList = [];
      }
    },

    onFactoryChange() {
      this.taskForm.causedDept = '';
      this.taskForm.owner = '';
      this.getDepartmentList();
     /* this.queryParams.processName = '';
      this.getProcessList();
      this.queryParams.measureMachine = '';
      this.getEnvList();
      this.initQuery();*/
    },
    onDeptChangeNew() {

      this.taskForm.owner = '';
      this.getMesEmployeeList();
     /* this.queryParams.processName = '';
      this.getProcessList();

      this.initQuery();*/
    },
    open() {
      this.visible = true;
      this.visibleSlot = true;
    },
    cancel() {
      this.visible = false;
    },
    confirm() {
      let cancel = () => this.cancel();
      this.ok(cancel);
      this.autoClose && cancel();
    },
    beClose(done) {
      done();
      this.beforeClose();
      this.cancel();
    },
    getDialogEvents() {
      // closed: () => this.visibleSlot = false是为了防止弹窗中的内容先于弹窗消失而造成弹窗在关闭时有一个突然向上缩小的情况
      let { close } = this.config || {}, events = { closed: () => this.visibleSlot = false };

      if (close && typeof close == 'function') {
        Object.assign(events, {
          close,
        })
      }

      return events
    },
    validateMeQeForm(fields) {
      let isError = false;
      this.$refs.peqeForm.validateField(fields, (errorMsg) => {
        if (errorMsg) {
          isError = true;
        }
      });
      return isError;
    },
    // 驳回
    backForm(data){
      // 驳回原因
      this.form.rejectReason =  data.rejectReason;
      this.form.button = 'back';
      applyInfo(this.form).then(response => {
        this.form.qeStatus = response.data.qeStatus;
        this.form.peStatus = response.data.peStatus;
        this.form.triggerStatus = response.data.triggerStatus;
        this.$modal.msgSuccess("操作成功");
        this.refreshList();
      });
    },
    dialogButton(button, handle) {
      if ('me' == handle) {
        let arr = ['peReason', 'taskList'];
        if(this.form.isMultiple == 'Y') {
          arr = ['peReason', 'taskList', 'checkAccessory' , 'checkTriggers'];
        }
        if (this.validateMeQeForm(arr)) {
          return false;
        }
        if(this.form.isMultiple == 'Y') {
          let now = new Date();
          let year = now.getFullYear();
          let month = (now.getMonth() + 1).toString().padStart(2, '0');
          let day = now.getDate().toString().padStart(2, '0');
          let hours = now.getHours().toString().padStart(2, '0');
          let minutes = now.getMinutes().toString().padStart(2, '0');
          let seconds = now.getSeconds().toString().padStart(2, '0');
          for(let obj of this.form.checkTriggers) {
            if (obj.checkResult) {
              obj.checkPeople = this.$store.state.user.nickName;
              obj.checkTime = `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
            }
          }
          this.form.checkTrigger = JSON.stringify(this.form.checkTriggers);
        }
      }
      if ('qe' == handle) {
        if (this.validateMeQeForm(['qeRisk', 'qeValidity'])) {
          return false;
        }
      }
      this.form.button = button;
      applyInfo(this.form).then(response => {
        this.form.qeStatus = response.data.qeStatus;
        this.form.peStatus = response.data.peStatus;
        this.form.triggerStatus = response.data.triggerStatus;
        this.$modal.msgSuccess("操作成功");
        this.refreshList();
      });
    },
    refreshList() {
      this.$emit('refreshList', true);
    },
    loadData() {
      postInfo(this.id).then(response=>{
        if (response && response.data) {
          this.title = '测量时间：'+response.data.triggerSpcTime.substr(0, 19) + ' 管制项目名称：' + response.data.spcName + ' 触发异常：' + response.data.triggerType
        }
      })
      this.tabClick({name:this.activeName});
      if(this.ocap){
        this.$refs.ocapBill.loadData({spcType: "CHEMICAL", exceptionId: this.id})
      }
      if(this.monitor){
        this.$refs.chemicalMonitor.loadData();
      }
    },
    setData() {
      let that = this
      setInterval(function () {
        for (var i = 0; i < 5; i++) {
          that.data.shift()
          that.data.push(that.randomData())
          that.data1.shift()
          that.data1.push(that.randomData1(400))
          that.xData.shift()
          that.xData.push(that.getXAxisData())
        }
        that.myChart.setOption({
          color: '#0000FF',
          xAxis: {
            data: that.xData
          },
          series: [
            {
              data: that.data
            },
            {
              data: that.data1
            }
          ]
        })
      }, 1000)
    },
    randomData() {

      let value = Math.random() * 1000
      this.now = new Date(+this.now + this.oneDay)
      value = value + Math.random() * 21 - 10

      return Math.round(value)

    },
    randomData1(v) {
      return v
    },
    tabClick(tab, event) {
      if (tab.name == 'peqe') {
        this.$refs.peqeForm.clearValidate();
        getInfo(this.id).then(response => {
          let updatedForm = response.data
          // 当页面切换时，当是同一个tab的切换保持缓存
          if (this.form.id === response.data.id) {
            updatedForm.peReason = this.form.peReason;
            updatedForm.qeRisk = this.form.qeRisk;
            updatedForm.qeValidity = this.form.qeValidity;
            updatedForm.accessory = this.form.accessory;
          }
          this.form = updatedForm;

          if (response.data.isMultiple == 'Y' && response.data.checkTrigger == null) {
            this.form.checkTriggers = [
              { num: '1', checkMsg: '本次异常与前次发生的异常是否同样的原因?', checkResult: '', checkPeople: '', checkTime: '' },
              { num: '2', checkMsg: '前次的异常改善对策是否已经有效实施?', checkResult: '', checkPeople: '', checkTime: '' },
              { num: '3', checkMsg: '前次异常的真因为什么再发?', checkResult: '', checkPeople: '', checkTime: '' }
            ];
          } else if (response.data.checkTrigger) {
            this.form.checkTriggers = JSON.parse(response.data.checkTrigger);
          }
        });
        // 获取task列表
        this.getTaskList();
      } else if (tab.name == 'history') {
        let queryParam = { pageNum: 1, pageSize: 200, type: 'CHEMICAL', execptionId: this.id };
        listLog(queryParam).then(response => {
          this.loglList = response.records;
          //this.total = response.total;
          //this.loading = false;
        });
      }
      // if(tab.name =='monitor'){
      //     let param = {id:this.id}

      // }
    },
    // 获取Task列表
    getTaskList() {
      getTaskList(this.id).then(response => {
        this.taskList = response;
      });
    },
    // 新增Task
    taskAdd() {
      this.taskOpen = true;
      this.taskTitle = '新增Task';
      this.taskForm = {
        owner: ""

      };
    },
    // 关闭Task弹窗
    taskCancel() {
      this.taskOpen = false;
      this.resetTask();
    },
    // 重置Task表单
    resetTask() {
      this.taskForm = { owner: ""};
      this.taskTitle = "编辑task";
      this.resetForm("taskForm");
    },
    // 驳回
    backTaskForm(data) {
      // 驳回原因
      this.taskForm.rejectReason =  data.rejectReason;
      this.taskForm.spcType = 'CHEMICAL';
      this.taskForm.exceptionId = this.id;
      if (this.form.triggerSpcTime) {
        this.taskForm.startDate = this.form.triggerSpcTime;
      }
      if (this.taskForm.id != undefined) {
        this.taskForm.submitStatus = 'back';
        this.taskForm.status = 'open';
        this.taskForm.finisher = '';
        this.taskForm.finisherTime = '';

        editTask(this.taskForm).then(response => {
          this.$modal.msgSuccess("操作成功");
          this.taskCancel();
          this.getTaskList();
        });
      } else {
        addTask(this.taskForm).then(response => {
          this.$modal.msgSuccess("新增成功");
          this.taskCancel();
          this.getTaskList();
        });
      }
    },

    // 提交Task
    submitTaskForm(button) {
      this.$refs["taskForm"].validate(valid => {
        if (valid) {
          this.taskForm.spcType = 'CHEMICAL';
          this.taskForm.exceptionId = this.id;
          if (this.form.triggerSpcTime) {
            this.taskForm.startDate = this.form.triggerSpcTime;
          }
          if (this.taskForm.id != undefined) {
            this.taskForm.submitStatus = button;
            if(this.form.triggerStatus === 'close'){
              this.taskForm.status = (button == 'close' ? button:'forward');
            }

            editTask(this.taskForm).then(response => {
              this.$modal.msgSuccess("操作成功");
              this.taskCancel();
              this.getTaskList();
            });
          } else {
            addTask(this.taskForm).then(response => {
              this.$modal.msgSuccess("新增成功");
              this.taskCancel();
              this.getTaskList();
            });
          }
        }
      });
    },
    // 修改Task
    taskUpdate(row) {
      this.resetTask();
      getTask(row.id).then(response => {
        this.taskForm = response.data;
        this.taskOpen = true;
        this.taskTitle = "修改Task";
      });
    },
    // 删除Task
    taskDelete(row) {
      this.$modal.confirm('是否确认删除该task数据项？').then(function () {
        return removeTask(row.id);
      }).then(() => {
        this.getTaskList();
        this.$modal.msgSuccess("删除成功");
      }).catch(() => { });
    }
  },
  mounted() {

  }
};
</script>

<style lang="scss">
.el-divider {
  margin-top: 0;
}

.left {
  float: left;
}

.right {
  float: right;
}

.module-line {
  padding-left: 5px;
  margin-bottom: 10px;
  border-left: 5px solid #409EFF;
}

.module-title {
  border-bottom: 1px solid #dfe4ed;
  padding: 10px 0px;
  font-weight: bold;
}

.el-form-item.long-label .el-form-item__label {
  line-height: 20px;
}

//.el-dialog-cus {
//  .el-dialog {
//    padding: 8px;
//  }
//  .el-dialog__title {
//    font-weight: bold;
//  }
//  .el-dialog__header {
//    padding: 20px 0 12px;
//  }
//  .el-dialog__headerbtn {
//    top: 8px;
//    right: 8px;
//  }
//  .el-dialog__body {
//    padding: 0 24px;
//  }
//  .el-dialog__footer {
//    padding: 20px;
//    .el-button {
//      padding: 8px 20px;
//      & + .el-button {
//        margin-left: 40px;
//      }
//    }
//  }
//}
</style>
