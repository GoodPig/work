<template> 
  <div class="cloud-component drugstoreDirectory">
    <div class="searchTitle">
      <span>药品目录</span>
    </div>
    <!-- 搜索区域 -->
    <div class="cloud-search el-form-item-small">
      <el-form :inline="true" :model="ruleForm" ref="ruleForm">
        <el-form-item label="药品名称：">
          <el-input v-model.trim="ruleForm.name"
                    placeholder="请输入药品名称" :clearable="true"></el-input>
        </el-form-item>
        <el-form-item class="ml30">
          <el-button type="primary" @click="search" size="mini" :loading="loading">查询</el-button>
          <el-button @click="reset" size="mini">重置</el-button>
          <span class="delimiter"></span>
          <el-button type="primary" @click="upload" size="mini">导入药品</el-button>
          <el-button @click="download" size="mini">导出模板</el-button>
        </el-form-item>
      </el-form>
      <div class="addButton">
        <el-button @click="add" size="mini">新增</el-button>
      </div>
    </div>
    <!--搜索结果-->
    <div class="cloud-search-list">
      <echarts-contain containType="big" :parentHeight="routerViewHeight" :heightRatio="1">
        <el-table :height="(dataList.content && dataList.content.length>0)?(routerViewHeight*1-85):(routerViewHeight*1)"
                  :data="dataList.content" style="width: 100%" v-loading="loading"
                  :empty-text="emptyText" :element-loading-text="elementLoadingText" stripe highlight-current-row>
          <el-table-column prop="code" label="药店药品编码" min-width="12%" show-overflow-tooltip>
          </el-table-column>
          <el-table-column prop="commonName" label="通用名" min-width="12%" show-overflow-tooltip>
          </el-table-column>
          <el-table-column prop="name" label="商品名" min-width="12%" show-overflow-tooltip>
          </el-table-column>
          <el-table-column prop="spec" label="单价" min-width="12%" show-overflow-tooltip>
          </el-table-column>
          <el-table-column prop="type" label="规格" min-width="12%" show-overflow-tooltip>
          </el-table-column>
          <el-table-column prop="manufacturer" label="剂型" min-width="12%" show-overflow-tooltip>
          </el-table-column>
          <el-table-column prop="manufacturer" label="厂家地址" min-width="16%" show-overflow-tooltip>
          </el-table-column>
          <el-table-column prop="status" label="状态" min-width="10%" show-overflow-tooltip>
            <template slot-scope="scope">
              <span class='redStatus' v-if="scope.row.status == 0">下架</span>
              <span class='greenStatus' v-else>上架</span>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="150px">
            <template slot-scope="scope">
              <el-button type="text" size="small" @click="changeStatus(scope.row)">
                <template>
                  <i class="icon-up" title="上架" v-if="scope.row.status == 0"></i>
                  <i class="icon-down" title="下架" v-else></i>
                </template>
              </el-button>
              <el-button type="text" size="small" @click="edit(scope.row)" style="margin-left: 8px">
                <i class="icon-edit" title="编辑"></i>
              </el-button>
              <el-button type="text" size="small" @click="deleteThis(scope.row.id)">
                <i class="icon-delete" title="删除"></i>
              </el-button>
            </template>
          </el-table-column>
        </el-table>
        <!-- 分页 -->
        <pagination :data="dataList" @change="getDataList"></pagination>
      </echarts-contain>
    </div>
    <!--新增/编辑药品弹框-->
    <el-dialog :title="dialogTitle" :visible.sync="dialogVisible" :append-to-body="true" width="350px" @close="cancelDrugDialog">
      <el-form :inline="true" :model="ruleFormDialog" ref="ruleFormDialog" :rules="dialogRules" label-width="75px" class="el-dialog--center">
        <el-form-item label="药品编码：" prop='code'>
          <el-input v-model.trim="ruleFormDialog.code"  placeholder="请输入药品编码" :maxlength="30" :clearable="true"></el-input>
        </el-form-item>
        <el-form-item label="通用名：" prop='commonName'>
          <el-input v-model.trim="ruleFormDialog.commonName"  placeholder="请输入药品通用名" :maxlength="30" :clearable="true"></el-input>
        </el-form-item>
        <el-form-item label="商品名：" prop='name'>
          <el-input v-model.trim="ruleFormDialog.name" placeholder="请输入药品商品名" :maxlength="30" :clearable="true"></el-input>
        </el-form-item>
        <el-form-item label="规格：" prop='spec'>
          <el-input v-model.trim="ruleFormDialog.spec" placeholder="例如：0.6g*24片/盒" :maxlength="30" :clearable="true"></el-input>
        </el-form-item>
        <el-form-item label="药品价格：" prop='price'>
          <el-input v-model.trim="ruleFormDialog.price"  placeholder="请输入药品药品价格" :maxlength="30" :clearable="true"></el-input>
        </el-form-item>
        <el-form-item label="剂型：" prop='type'>
          <el-input v-model.trim="ruleFormDialog.type" placeholder="例如：胶囊剂、片剂" :maxlength="30" :clearable="true"></el-input>
        </el-form-item>
        <el-form-item label="生产厂家：" prop='manufacturer'>
          <el-input v-model.trim="ruleFormDialog.manufacturer" placeholder="请输入药品生产厂家" :maxlength="30" :clearable="true"></el-input>
        </el-form-item>
        <div class="el-dialog--center">
          <el-button @click="cancelDrugDialog" size="mini">取消</el-button>
          <el-button type="primary" @click="saveDrugDialog" size="mini">确定</el-button>
        </div>
      </el-form>
    </el-dialog>

  </div>
</template>

<script>
  import utils from 'components/utils'
  import echartsContain from 'components/packages/echartsContain/echartsContain'
  import {pageSize, pageNo, emptyText, elementLoadingText} from 'components/utils/constant'
  import pagination from 'components/packages/pagination/pagination'

  export default {
    name: 'drugstoreDirectory',
    mixins: [],
    data() {
      return{
        ruleForm:{
          name:'',
        },
        params_ruleForm:{
          name:'',
        },
        dataList: {},
        loading: false,
        pageNo: '',
        pageSize: '',
        currentPageNo: '',
        currentPageSize: '',
        emptyText: '',
        elementLoadingText: '',

        dialogTitle:'',
        dialogVisible:false,
        ruleFormDialog:{
          code:'',
          commonName:'',//药品通用名
          name:'',//药品名称
          spec:'',//药品规格
          type:'',//剂型
          price:'',
          manufacturer:'',//生产厂商
          hospitalId:'',
          hospitalName:'',
          id:''
        },
        dialogRules: {
          code:[
            {required: true, message:'药品编码不能为空！',trigger: 'blur'}
          ],
          commonName:[
            {required: true, message:'药品通用名不能为空！',trigger: 'blur'}
          ],
          name:[
            {required: true, message:'药品名称不能为空！',trigger: 'blur'}
          ],
          spec:[
            {required: true, message:'药品规格不能为空！',trigger: 'blur'}
          ],
          price:[
            {required: true, message:'药品价格不能为空！',trigger: 'blur'}
          ],
          type:[
            {required: true, message:'剂型不能为空！',trigger: 'blur'}
          ],
          manufacturer:[
            {required: true, message:'生产厂商不能为空！',trigger: 'blur'}
          ]
        }
      }
    },
    watch: {},
    computed: {},
    created() {
      this.initPage()
    },
    mounted() {

    },
    components: {
      pagination,
      echartsContain
    },
    methods: {
      upload(){

      },
      download(){
        let that = this;
        let hospitalName = '医院药品目录.xls';
        that.$axios({
          method: 'get',
          headers: {'Authorization':that.token},
          url: './peps/drug/hospital/download',
        }).then((response) => {
          let blob = new Blob([response.data])   
          that.$download(hospitalName, blob)
        })
      },
      initPage(){
        this.$emit('handlePageHeight');//初始化的时候首先调用调整窗口
        this.pageNo = pageNo;
        this.pageSize = pageSize;
        this.emptyText = emptyText;
        this.elementLoadingText = elementLoadingText;
        this.userLogin = JSON.parse(sessionStorage.getItem('CURR_USER_PRESCRIPTION_SUPERVISE')).userLogin;
        this.userInfo = JSON.parse(localStorage.getItem('userInfo'));
        this.token = this.userLogin.data
        this.search();
      },
      search(){
        this.params_ruleForm = utils.deepClone(this.ruleForm);
        this.$nextTick(() => {
          this.getDataList()
        });
      },
      reset(){
        //区域对整个表单进行重置，将所有字段值重置为初始值并移除校验结果
        this.$refs.ruleForm.resetFields();
        //初始化注册data
        Object.assign(this.$data, this.$options.data());
        //初始化data默认值及页面
        this.initPage()
      },
      getDataList(pageNo = this.pageNo, pageSize = this.pageSize) {

        let that = this;
        that.currentPageNo = pageNo;
        that.currentPageSize = pageSize;
        that.loading = true;
        let formData = {
          "pageIndex": pageNo * 10 - 10,
          "itemCountPerPage": pageSize,
          args:{
            "name": that.params_ruleForm.name,
          }
        };
        that.$axios({
          method: 'post',
          headers: {'Authorization':this.token},
          url: './peps/drug/hospital/search/page?t=' + (+new Date()),
          data: formData
//            that.removeNullKey(formData)
        }).then((response) => {
          console.log(response)
          that.loading = false;
          if (response.data.code == 0) {
            let obj = {};
            obj.content = response.data.data.list;
            obj.pageNo = pageNo;
            obj.pageSize = pageSize;
            obj.totalCount = parseInt(response.data.data.total);
            obj.totalPage = parseInt((obj.total + obj.pageSize - 1) / obj.pageSize);
            that.dataList = obj;
          }
          else {
            that.$warning();
          }
        }).catch((error) => {
          that.loading = false;
          console.log(error)
        });
      },
      add(){
        this.dialogTitle = "添加药品";
        this.dialogVisible = true;
        this.drugOption = 'add';

      },
      edit(row){
        this.dialogTitle = "编辑药品";
        this.dialogVisible = true;
        this.drugOption = 'edit';
        // this.disabledFlag = true;
        for(let key in row){
          this.ruleFormDialog[key] = row[key];
        }
      },
      saveDrugDialog(){
        // this.dialogVisible = false;

        let that = this;
        that.$refs.ruleFormDialog.validate((valid) => {
          if (!valid) {
            return false;
          }
          if(that.drugOption === 'add'){
            var formData = {
              code:that.ruleFormDialog.code,//药品编码
              commonName: that.ruleFormDialog.commonName,//药品通用名
              name: that.ruleFormDialog.name,//药品名称
              spec: that.ruleFormDialog.spec,//药品规格
              type: that.ruleFormDialog.type,//剂型
              manufacturer: that.ruleFormDialog.manufacturer,//生产厂商
              price:that.ruleFormDialog.price,//单价
              hospitalId:that.userInfo.orgId,
              hospitalName:that.userInfo.orgName,
              hospitalCode:that.userInfo.orgCode
            };

            that.saveUrl = './peps/drug/hospital/add';
          }
          else{
            var formData = {  
              filter: {
                id:that.ruleFormDialog.id,//药品id
              },
              record: {
                code:that.ruleFormDialog.code,//药品编码
                commonName: that.ruleFormDialog.commonName,//药品通用名
                name: that.ruleFormDialog.name,//药品名称
                spec: that.ruleFormDialog.spec,//药品规格
                type: that.ruleFormDialog.type,//剂型
                manufacturer: that.ruleFormDialog.manufacturer,//生产厂商
                price: that.ruleFormDialog.price
              },
            };
            that.saveUrl = './peps/drug/hospital/update?t=' + (+new Date());
          }
          that.$axios({
            method: 'post',
            headers: {'Authorization':that.token},
            url: that.saveUrl,
            data: formData
          }).then((response) => {
            let data = response.data;
            if (data.code == '0') {
              that.$message({
                type:'success',
                message:'添加成功'
              })
              that.dialogVisible = false;
              that.getDataList()
            }else {
              that.$message({
                message: data.message,
                type: 'notice'
              });
            }
          }).catch((error) => {
            console.log(error)
          });

        })



      },
      cancelDrugDialog(){
        this.dialogVisible = false;
      },
      changeStatus(row){
        let that = this;
        let tips = '';
        if(row.status == 1){
          tips = '下架';
        }else{
          tips = '上架';
        }
        that.$confirm('确定是否'+ tips +'?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type:'warning'
        }).then(() => {
          let formData = {
            id : row.id,
            status: row.status == 0?1:0,
          };
          that.$axios({
            method: 'get',
            headers: {'Authorization':this.token},
            url: './peps/drug/hospital/onSale?t=' + (+new Date()),
            params: formData
          }).then((response) => {
            if (response.data.code == 0) {
              this.search();
              console.log(response)
            }else{
              that.$warning("修改状态失败");
            }
          }).catch((error) => {
            console.log(error)
          });
        }).catch({

        });

      },
      deleteThis(row){
        let that = this
        let tips = '删除此药品信息'
        that.$confirm('确定是否'+ tips +'?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type:'warning'
        }).then(() => {
          let formData = {
            id : row
          };
          that.$axios({
            method: 'get',
            headers: {'Authorization':this.token},
            url: './peps/drug/hospital/delete',
            params: formData
          }).then((response) => {
            if (response.data.code == 0) {
              this.search();
              console.log(response)
            }else{
              that.$warning("修改状态失败");
            }
          }).catch((error) => {
            console.log(error)
          });
        }).catch({

        });
      },
     





    },
    beforeRouteEnter(to, from, next) {

      next()
    },
    beforeRouteLeave(to, from, next) {

      next()
    },
  }

</script>

<style scoped>

</style>

<style>
  .drugstoreDirectory .el-form .el-form-item .el-input{
    width:220px
  }
  .el-dialog .el-form .el-form-item .el-input{
    width:220px
  }

</style>
