<template>
  <div class="dashboard-container">

    <el-row :gutter="24">
      <el-col :span="leftchangge">
        <div class="grid-content bg-purple">
          <el-card class="box-card">

            <div slot="header" class="clearfix">
              <el-button  type="primary" size="medium"
                          :loading="loadingSave" @click="SaveTest"
                          icon="el-icon-tickets">保存接口</el-button>
              <el-button  plain size="medium" @click="fastTest" :loading="loadingSend">模拟请求</el-button>
              <el-button @click="leftchage" v-show="!rightstatus" >查看结果
                <i class="el-icon-d-arrow-right"></i><i class="el-icon-d-arrow-right"></i>
              </el-button>
              <el-button type="info" icon="el-icon-info"
                         style="float: right" size="medium" @click="getinfo">函数</el-button>
            </div>
            <div style="height:100%;overflow:auto;height: 80vh;">
              <el-form :model="form" ref="form" :rules="formRules" label-width="100px">

                <el-form-item label="所属项目：" prop="selectedOptions3">
                  <el-cascader
                    placeholder="搜索：  用户项目"
                    :options="options"
                    v-model="form.selectedOptions3"
                    @change="handleOptionsChange"
                    filterable
                    change-on-select></el-cascader>
                </el-form-item>
                <el-form-item label="接口名称：" prop="name">
                  <el-input v-model="form.name" placeholder="请输入接口名称"></el-input>
                </el-form-item>
                <el-form-item label="请求方法：" prop="methods">
                  <el-select v-model="form.methods" placeholder="Method" @change="checkRequest">
                    <el-option v-for="(item,index) in methods" :key="index+''" :label="item.label" :value="item.value"></el-option>
                  </el-select>
                </el-form-item>
                <el-form-item label="请求地址：" prop="addr">
                  <el-input type="textarea" v-model="form.addr" :rows="4" placeholder="请输入请求地址"></el-input>
                </el-form-item>

                <el-row :span="24">

                  <el-form-item label="请求头部：">
                    <el-table :data="form.head" highlight-current-row ref="multipleHeadTable">
                      <el-table-column prop="name" label="Name" min-width="150">
                        <template slot-scope="scope">
                          <el-input  v-model.trim="scope.row.name" :value="scope.row.name" placeholder="请输入标签"></el-input>
                        </template>
                      </el-table-column>
                      <el-table-column prop="value" label="Value" min-width="180">
                        <template slot-scope="scope">
                          <el-input v-model.trim="scope.row.value" :value="scope.row.value" placeholder="请输入内容"></el-input>
                        </template>
                      </el-table-column>
                      <el-table-column label="操作" min-width="50">
                        <template slot-scope="scope">
                          <i class="el-icon-delete" style="font-size:20px;cursor:pointer;" @click="delHead(scope.$index)"></i>
                        </template>
                      </el-table-column>
                      <el-table-column label="" min-width="60">
                        <template slot-scope="scope">
                          <el-button v-if="scope.$index===(form.head.length-1)" size="mini" class="el-icon-plus" @click="addHead"></el-button>
                        </template>
                      </el-table-column>
                    </el-table>
                  </el-form-item>

                  <el-form-item label="请求参数：">
                    <el-row :span="24">
                      <el-tabs v-model="activeName2" type="card">
                        <el-tab-pane label="Parameters" name="form">
                          <el-table ref="multipleParameterTable" :data="form.parameter"
                                    highlight-current-row
                                    @selection-change="selsChangeParameter">
                            <el-table-column prop="name" label="Name" min-width="150">
                              <template slot-scope="scope">
                                <el-input v-model.trim="scope.row.name" :value="scope.row.name" placeholder="请输入参数名"></el-input>
                              </template>
                            </el-table-column>
                            <el-table-column prop="value" label="Value" min-width="180">
                              <template slot-scope="scope">
                                <el-input v-model.trim="scope.row.value" :value="scope.row.value" placeholder="请输入参数值"></el-input>
                              </template>
                            </el-table-column>
                            <el-table-column label="操作" min-width="50">
                              <template slot-scope="scope">
                                <i class="el-icon-delete" style="font-size:30px;cursor:pointer;" @click="delParameter(scope.$index)"></i>
                              </template>
                            </el-table-column>
                            <el-table-column label="" min-width="60">
                              <template slot-scope="scope">
                                <el-button v-if="scope.$index===(form.parameter.length-1)" size="mini" class="el-icon-plus" @click="addParameter"></el-button>
                              </template>
                            </el-table-column>
                          </el-table>
                        </el-tab-pane>
                        <el-tab-pane label="Body Data" name="raw">
                          <template>
                            <el-input type="textarea" :rows="7" placeholder="请输入请求参数" v-model.trim="form.parameterRaw"></el-input>
                          </template>
                        </el-tab-pane>
                      </el-tabs>
                    </el-row>
                  </el-form-item>
                </el-row>

              </el-form>
            </div>


          </el-card>

        </div>
      </el-col>

      <el-col :span="12" v-show="rightstatus">
          <div class="grid-content bg-purple">
            <el-card class="box-card">
              <div style="z-index: 9;float: right;position: relative;">
                <el-button  plain size="medium" @click="rightclick">关闭</el-button>
              </div>
              <el-tabs v-model="activeName1" type="card" >
                <el-tab-pane label="响应内容" name="res">
                  <div v-if="!resultShow">
                    暂无响应内容
                  </div>
                  <div v-else="resultShow" style="height: 75vh;overflow: auto">
                    状态码：<pre>{{form.res_code}}</pre>
                    响应cookies：<pre>{{form.resultCookies}}</pre>
                    响应内容：<pre>{{form.res_body}}</pre>
                  </div>
                </el-tab-pane>
                <el-tab-pane label="请求内容" name="req">
                  <div v-if="!resultShow">
                    暂无请求内容
                  </div>
                  <div v-else="resultShow" style="height: 75vh;overflow: auto">
                    请求URL: <pre>{{form.req_url}}</pre>
                    请求头部：<pre>{{ form.req_headers }}</pre>
                    请求cookies：<pre>{{ form.req_cookies }}</pre>
                    请求参数：<pre>{{ form.req_body }}</pre>
                  </div>
                </el-tab-pane>
              </el-tabs>

            </el-card>
          </div>
      </el-col>

    </el-row>

    <el-dialog title="函数列表" :visible.sync="dialogVisible" width="40%">
          <template>
            <el-table
              :data="Functions"
              style="width: 100%"
              height="40vh">
              <el-table-column
                label="Name"
                prop="variableName">
              </el-table-column>
              <el-table-column
                label="Value"
                prop="variableValue">
              </el-table-column>
              <el-table-column
              label="操作">
                <template slot-scope="scope">
                  <el-button type="text" size="medium"
                             v-clipboard:copy="scope.row.variableValue"
                             v-clipboard:success="onCopy"
                             v-clipboard:error="onError">复制
                  </el-button>
                </template>

              </el-table-column>
            </el-table>
          </template>
    </el-dialog>

  </div>
</template>

<script>

  export default {
    name: 'Dashboard',
    data() {

      const validProjectName = (rule, value, callback) => {
        if(!this.load){
          if(!value){
            callback(new Error('请输入接口名称'))
          }else {
            callback()
          }
        }else {
          callback()
        }
      }
      const validProject = (rule, value, callback) => {
        if(!this.load){
          if(value.length === 0){
            callback(new Error('请选择所属项目'))
          }else {
            callback()
          }
        }else {
          callback()
        }
      }
      const validHttp =(rule,value,callback)=>{
        if(!value){
          callback(new Error('请输入URL地址'))
        }else if(value.indexOf("http://") !==0 && value.indexOf("https://") !==0){
          callback(new Error('请输入正确的URL地址'))
        }else{
          callback()
        }

      }

      return {
        dialogVisible: false,
        activeName1: 'res',
        activeName2: 'form',
        leftwidth: 24,
        rightstatus: false,
        options: [],
        methods: [
          {value: 'GET', label: 'GET'},
          {value: 'POST', label: 'POST'},
          // {value: 'put', label: 'PUT'},
          // {value: 'delete', label: 'DELETE'}
        ],
        ParameterType: true,
        load:true,
        loadingSend: false,
        loadingSave: false,
        result: true,
        activeNames: ['1', '2', '3', '4'],
        formchange: true,
        form: {
          selectedOptions3: [],
          url:"",
          name: '',
          methods: 'GET',
          addr: '',
          head: [
            {name: "", value: ""}
          ],
          parameterRaw: "",
          parameter: [
            { name: "", value: ""}
          ],
          paramstype: "",
          statusCode: "",
          resultData: "",
          resultHead: "",
          resultCookies: ""
        },
        formRules: {
          selectedOptions3: [
            { required: true, validator: validProject,trigger:'blur'}
          ],
          name: [
            { required: true, validator: validProjectName,trigger:'blur'},
            { min: 2, max: 30, message: '请输入长度在2到30个字符的接口名称', trigger: 'blur'}
          ],
          addr: [
            { required: true, message: '请输入URL地址', trigger: 'blur' },
            { required: true, validator: validHttp, trigger: 'blur' },
          ]
        },
        project_id: "",
        case_id: "",
        api_id: '',
        headers: "",
        parameters: "",
        resultShow: false,
        format: true,
        reqheaders: "",
        reqbody: "",
        reqaddr: "",
        Functions:[],
      }
    },
    methods: {
      // 复制成功
      onCopy(e){
        // console.log(e);
      },
      // 复制失败
      onError(e){
        // alert("失败");
      },
      getinfo(){
        this.$axios.get('/api/variable_list')
          .then(res => {
              if(res.data.status === 'SUCCESS'){
                this.Functions = res.data.data
              }else {
                this.$message.info(res.data.msg)
              }

          })
          .catch(error => {
            console.log(error)
          })
        this.dialogVisible = true
      },
      leftchage(){
        this.leftwidth = 12
        this.rightstatus = true

      },
      rightclick(){
        this.rightstatus = false
        this.leftwidth = 24
      },
      toJSON(str) {
        try {
          if (typeof JSON.parse(str) === "object") {
            return JSON.parse(str);
          }else{
            return str;
          }
        } catch(e) {
          return str;
        }
      },
      getApi(){
        this.$axios.post('/api/detail', this.api_id)
          .then(response => {
            if (response.data.status === 'SUCCESS') {
              this.form.name = response.data.data.name
              this.form.methods = response.data.data.method
              this.form.addr = response.data.data.url
              if(!!response.data.data.headers){
                console.log(response.data.data.headers)
                var head_obj = JSON.parse(response.data.data.headers)
                let head = []
                for(var i=0;i<Object.keys(head_obj).length;i++) {
                  for(var item in head_obj){
                    let hd = new Object();
                    hd.name = item
                    hd.value = head_obj[item]
                    head.push(hd)
                  }
                }
                this.form.head = head
              }else{
                this.form.head = [{name: "", value: ""}]
              }

              if(!!response.data.data.body){
                if(response.data.data.paramstype === 'raw'){
                  this.activeName2 = 'raw'
                  this.form.parameterRaw = response.data.data.body
                }else{
                  this.activeName2 = 'form'
                  var bd_obj = JSON.parse(response.data.data.body)
                  let bd = []
                  for(var i=0;i<Object.keys(bd_obj).length;i++) {
                    for(var item in bd_obj){
                      let hd = new Object();
                      hd.name = item
                      hd.value = bd_obj[item]
                      bd.push(hd)
                    }
                  }
                  this.form.parameter = bd
                }
              }else{
                this.activeName2 = 'form'
              }
            }
          })
          .catch(error => {
            console.log(error)
            this.activeName2 = 'form'
            this.$message.error("获取API详情失败")
          })
      },
      getApiTree(){
        this.$axios.post('/case/list')
          .then(response => {
            if (response.data.status === 'SUCCESS') {
              this.options = response.data.data
              // console.log(this.options)
            }else{
              this.options = [];
              this.$message.error("获取测试集失败")
            }

          })
          .catch(error => {
            this.options = [];
            this.$message.error("获取测试集失败")
          })
      },
      handleOptionsChange(val){
        this.project_id = val[0]
        if(val.length === 2){
          this.case_id = val[1]
        }
      },
      checkRequest(){
        let request = this.form.methods;
        if (request==="GET" || request==="DELETE"){
          this.formchange=false
        } else {
          this.formchange=true
        }
      },
      toggleHeadSelection(rows) {
        rows.forEach(row => {
          this.$refs.multipleHeadTable.toggleRowSelection(row, true);
        });
      },
      toggleParameterSelection(rows) {
        rows.forEach(row => {
          this.$refs.multipleParameterTable.toggleRowSelection(row, true);
        });
      },
      selsChangeHead: function (sels) {
        this.headers = sels
      },
      selsChangeParameter: function (sels) {
        this.parameters = sels
      },
      getheaders(){
        let _headers = new Object();
        for (let i = 0; i < this.form.head.length; i++) {
          var a = this.form.head[i]["name"];
          if (a) {
            _headers[a] = this.form.head[i]["value"]
          }
        }
        if(_headers){
          _headers = JSON.stringify(_headers)
        }
        this.reqheaders = _headers;
        return _headers;
      },
      getparams(){
        let _parameter = new Object();
        let _type = this.activeName2;
        if (_type === 'form') {
          for (let i = 0; i < this.form.parameter.length; i++) {
            var a = this.form.parameter[i]["name"];
            if (a) {
              _parameter[a] = this.form.parameter[i]["value"];
            }
          }
          if(_parameter){
            _parameter = JSON.stringify(_parameter)
          }
          this.form.paramstype = "form"
        }
        if (this.form.parameterRaw && _type === "raw") {
          _parameter = this.form.parameterRaw
          this.form.paramstype = "raw"
        }
        this.reqbody = _parameter;
        return _parameter
      },
      commonTest(url){
        this.$refs.form.validate((valid) => {
          if (valid) {
            if(this.load){
              this.loadingSend = true
            }else{
              this.loadingSave = true
            }
            let _headers = this.getheaders();
            let _parameter = this.getparams();
            this.form.statusCode = '';
            this.form.resultData = '';
            this.form.resultHead = '';
            this.form.resultCookies = '';

            this.project_id = this.form.selectedOptions3[0]
            if(this.form.selectedOptions3.length === 2){
              this.case_id = this.form.selectedOptions3[1]
            }

            this.$axios.post(url, {
              'id': this.api_id,
              'project_id': this.project_id,
              'case_id': this.case_id,
              'name':this.form.name,
              'method': this.form.methods,
              'url':  this.form.addr,
              'headers': _headers,
              'body': _parameter,
              'paramstype': this.activeName2
            }).then(response => {
              this.loadingSend = false
              this.loadingSave = false
              if(response.data.id){
                this.$confirm(response.data.msg + '，是否跳转分组页面？', '提示', {
                  confirmButtonText: '确定',
                  cancelButtonText: '取消',
                  type: 'warning'
                }).then(() => {

                  this.$router.push({
                    path: '/api',
                    name: '测试分组',
                    query : {
                      project_id: this.project_id,
                      case_id: this.case_id
                    }
                  })

                }).catch((error) => {
                  console.log(error)
                });

              }else {
                this.form.res_code = response.data.data.res_code
                this.form.req_url = response.data.data.req_url
                this.form.req_headers = response.data.data.req_headers
                this.form.req_cookies = response.data.data.req_cookies
                this.form.req_body = response.data.data.req_body
                this.form.res_headers = this.toJSON(response.data.data.res_headers)
                this.form.res_body = this.toJSON(response.data.data.res_body)
                this.form.res_cookies = this.toJSON(response.data.data.res_cookies)
                this.activeName1 = 'res'
                this.resultShow = true
                this.leftchage()
              }

            }).catch(error => {

              this.loadingSend = false
              this.form.statusCode = error.code
              this.form.resultHead = error.headers
              this.form.resultData = error

            })
          }
        })
      },
      fastTest: function() {
        this.load = true
        this.commonTest('/api/test')
      },
      SaveTest(){
        this.load = false
        this.commonTest('/api/save')
      },
      neatenFormat() {
        /*let demo = document.getElementsByTagName('pre')[0];
        console.log(demo)
        hljs.highlightBlock(demo);
        this.format = !this.format*/
      },
      addHead() {
        let headers = {name: "", value: ""};
        this.form.head.push(headers);
        let rows = [this.form.head[this.form.head.length-1]];
        this.toggleHeadSelection(rows)
      },
      addParameter() {
        let headers = {name: "", value: "", required:"True", restrict: "", description: ""};
        this.form.parameter.push(headers);
        let rows = [this.form.parameter[this.form.parameter.length-1]];
        this.toggleParameterSelection(rows)
      },
      delHead(index) {
        if (this.form.head.length !== 1) {
          this.form.head.splice(index, 1)
        }else{
          this.form.head = [{ name: "", value: ""}]
        }
      },
      delParameter(index) {
        if (this.form.parameter.length !== 1) {
          this.form.parameter.splice(index, 1)
        }else {
          this.form.parameter = [{ name: "", value: ""}]
        }
      },
      // addResponse() {
      //   let headers = {name: "", value: "", required:"True", restrict: "", description: ""};
      //   this.form.response.push(headers)
      // },
      // delResponse(index) {
      //   if (this.form.response.length !== 1) {
      //     this.form.response.splice(index, 1)
      //   }
      // },
      parseParams(data) {
        try {
          var tempArr = [];
          for (var i in data) {
            var key = encodeURIComponent(i);
            var value = encodeURIComponent(data[i]);
            tempArr.push(key + '=' + value);
          }
          var urlParamsStr = tempArr.join('&');
          return urlParamsStr;
        } catch (err) {
          return '';
        }
      },
      handleChange(val) {
      },
      onSubmit() {
        // console.log('submit!');
      },
    },
    computed:{
      leftchangge(){
        return this.leftwidth
      }
    },
    created () {
      if(this.$route.query.project_id) {
        this.form.selectedOptions3 = [this.$route.query.project_id]
        if(this.$route.query.case_id){
          this.form.selectedOptions3 = [this.$route.query.project_id,this.$route.query.case_id ]
        }
      }
      if(this.$route.query.id){
        this.api_id = this.$route.query.id
        this.getApi()
      }
    },
    mounted() {
      this.getApiTree();
      this.toggleHeadSelection(this.form.head);
      this.toggleParameterSelection(this.form.parameter);
    },
  }
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
  .dashboard {
    &-container {
      margin: 15px 20px 15px 20px;
    }
  }
  .box-card {
    width: 100%;
    height: 90vh;
    /deep/ .el-card__body {
      padding: 10px;
      height: 80vh;
    }
  }
  .return-list {
    margin-top: 0px;
    margin-bottom: 10px;
  }
  .head-class {
    font-size: 17px
  }
  .parameter-a {
    display: block;
  }
  .parameter-b {
    display: none;
  }
  .el-table {
    /deep/ th{
      padding: 0px 0;
    }

  }
  pre {
    overflow-x: auto;
    white-space: pre-wrap;
    word-wrap: break-word;
  }

</style>



