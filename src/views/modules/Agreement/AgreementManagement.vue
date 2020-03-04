<template>
  <!-- 人员统计 -->
  <div class="mod-role">
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-input style="width:200px" v-model="dataForm.name" placeholder="设备类名称" clearable></el-input>
        <!-- <el-input style="width:200px" v-model="dataForm.keyword" placeholder="角色名称" clearable></el-input> -->
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('sys:role:save')" type="primary" @click="add()">新增</el-button>
      </el-form-item>
    </el-form>
    <el-table :data="dataList" border v-loading="dataListLoading" @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <!-- <el-table-column type="selection" header-align="center" align="center" width="50">
          </el-table-column> -->
      <el-table-column prop="name" header-align="center" align="center" width="180" label="设备类名称">
      </el-table-column>
      <el-table-column prop="status" header-align="center" align="center" :formatter="sexFormatter" width="180" label="是否显示">
      </el-table-column>

      <el-table-column header-align="center" align="center" width="180" label="其他信息">
        <template slot-scope="scope">
          <el-button v-if="isAuth('sys:role:update')" type="success" style="width:100px" @click="look(scope.row)">查看
          </el-button>
        </template>
      </el-table-column>
      <el-table-column prop="creatdate" header-align="center" align="center" min-width="280" label="操作时间">
      </el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="150" label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('sys:role:update')" type="text" size="small" @click="updateHandle(scope.row)">
            修改</el-button>
          <span>|</span>
          <el-button v-if="isAuth('sys:role:delete')" style="color:red" type="text" size="small"
            @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination @size-change="sizeChangeHandle" @current-change="currentChangeHandle" :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]" :page-size="pageSize" :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <el-dialog :title="title" :visible.sync="dialogFormVisible">
      <el-form :model="addForm">
        <el-form-item label="设备类名称" :label-width="formLabelWidth">
          <el-input style="width:90%" v-model="addForm.name" autocomplete="off"></el-input>
        </el-form-item>
        <!-- <el-form-item label="协议模板" :label-width="formLabelWidth">
          <el-select style="width:90%" v-model="addForm.region" placeholder="请选择协议模板">
            <el-option v-for="item in xieyiList" :key="item.id" :label="item.name" :value="item.id"></el-option>
          </el-select>
        </el-form-item> -->
        <el-form-item label="是否显示" :label-width="formLabelWidth">
          <el-radio-group v-model="addForm.status">
            <el-radio label="0">显示</el-radio>
            <el-radio label="1">不显示</el-radio>
          </el-radio-group>
        </el-form-item>

        <el-form-item label="上传照片" :label-width="formLabelWidth">
          <el-upload ref='upload' style="width:90%" :auto-upload='false' :file-list="fileList" :multiple='false'
            :limit="1" :http-request="uploadFiles" accept="image/jpeg,image/gif,image/png" action='' list-type="picture"
            :on-change='changeUpload'>
            <el-button slot="trigger" size="mini" type="primary">选取图片</el-button>
            <span>&nbsp;</span>
            <el-button @click='uploadFiles' size="mini" type="primary">点击上传</el-button>
          </el-upload>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button v-if="!ishealthstatus" @click="goback()">返 回</el-button>
        <el-button @click="cancel()">取 消</el-button>
        <el-button type="primary" @click="commit()">确 定</el-button>
      </div>
    </el-dialog>

        <el-dialog title="照片查看" :visible.sync="dialogimg" >
          <img style="width:60%;height:670px;margin-left:50%;transform: translate(-50%);" :src="imgurl" alt="">
    </el-dialog>
  </div>
</template>

<script>

  export default {
    data() {
      return {
        imgurl:"",
        dialogimg:false,
        url: "",
        title: "新增人员",
        ishealthstatus: true,
        FFFhealthstatus: "",
        // 0，无异常 1，发热 2乏力 3干咳 4 鼻塞 5流涕 6咽痛 7腹泻
        roleList: [{
          roleName: "发热",
          roleId: "1",
          id: 1,
        }, {
          roleName: "乏力",
          roleId: "2",
          id: 1,
        }, {
          roleName: "干咳",
          roleId: "3",
          id: 1,
        }, {
          roleName: "鼻塞",
          roleId: "4",
          id: 1,
        }, {
          roleName: "流涕",
          roleId: "5",
          id: 1,
        }, {
          roleName: "咽痛",
          roleId: "6",
          id: 1,
        }, {
          roleName: "腹泻",
          roleId: "7",
          id: 1,
        }],
        multipleSelection: [],
        beizhu: "",
        gridData: [],
        sendData: {},
        sssss: [],
        cascader: ["30"],
        props: {
          multiple: false,
          label: 'gName',
          value: "gCode",
        },
        options: [], //树形结构
        checkedRoleIds: [30],
        defaultProps: {
          children: 'children',
          label: 'label'
        },
        data: [],
        dialogFormVisible: false,
        xieyiList: [],
        formLabelWidth: '180px',
        dataForm: {
          name: '',
        },
        gcode: [],
        addForm: {
          name: "",

          status: "",

          image: ""
        },
        healthstatus: [],
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        selectedList: [],
        isId: "",
        editData: {},
        tem: [],
        no: true,
        fileList: [],
        rep: "",
      }
    },
    watch: {
      "FFFhealthstatus": function (val) {
        if (val == 1) {
          // this.ishealthstatus = false
        }
      }
    },
    activated() {
      this.getDataList()

    },
    methods: {
      look(val) {
        // console.log("]]]]",val.fileurl)
        let tem = val.fileurl.replace(this.rep, "http://192.168.0.185:8888")
        console.log(tem)
        this.imgurl = tem
        this.dialogimg = true
      },
      changeUpload(val) {
        console.log("val--", val)
        //  this.addForm.image.url
      },
      uploadFiles() {
        let file = this.$refs.upload.$refs['upload-inner'].$refs.input; //获取文件数据
        let fileList = file.files;
        let reader = new FileReader(); //html5读文件
        console.log("fileList[0]", fileList)
        console.log("fileList.size----", fileList[0].size <= 4194304);
        if (fileList[0].size >= 4194304) {
          this.$message({
            message: "图片大小不能超过4M",
            type: 'warning'
          });
          return;
        }
        let type = fileList[0].name.split(".")[1]
        // console.log(type)
        // if (type != "jpg" || type != "JPG" || type != "PNG"  || type != "png" || type != "jpeg") {
        //   this.$message({
        //     message: "您上传都格式有误，请重新上传",
        //     type: 'warning'
        //   });
        //   return
        // }
        let myform = new FormData()
        myform.append('file', fileList[0]);
        console.log("开始请求")
        this.$http({
          url: this.$http.adornUrl('tools/device/upload'),
          method: 'POST',
          data: myform
        }).then((data) => {
          console.log("返回值-----", data)
          if (data.data.code == 0) {
            this.$message({
              message: "上传成功",
              type: 'success'
            });
            this.addForm.image = data.data.res
            console.log("------this.addForm.image----", this.addForm.image)
          } else {
            this.$message({
              message: "上传失败",
              type: 'warning'
            });
          }
        })
        // reader.readAsDataURL(fileList[0]); //转BASE64  
        // let imgFile
        // // let that = this
        // reader.onload = e => {
        //     let imgFile
        //     imgFile = e.target.result;
        //     console.log("本地路径---",reader.result[0])
        //     // let arr = imgFile.split(',')
        //     // this.datas.faceBase64 = arr[1]
        //     // console.log(imgFile)
        //     this.formLabelAlign.image = imgFile
        //     if (this.formLabelAlign.image != "") {
        //         this.$message({
        //             message: "上传成功",
        //             type: 'success'
        //         });
        //     } else {
        //         this.$message({
        //             message: "上传失败",
        //             type: 'warning'
        //         });
        //     }
        // }
      },
      uploadSuccess(data) {
        console.log("--上传照片成功回调--", data)
      },
      goback() {
        //  this.ishealthstatus = !this.ishealthstatus
        this.FFFhealthstatus = "0"
      },
      cancel() {
        this.dialogFormVisible = false
      },
      sexFormatter(val) {
        if (val.status == 0) {
          return "显示"
        } else {
          return "不显示"
        }
      },
      idcardFormatter(val) { //身份证脱敏
        let temp = val.idcard
        let NewStr = String(val.idcard.substr(0, 6) + "****" + val.idcard.substr(6 + "****".length))
        return NewStr.substr(0, 14) + "****" + NewStr.substr(14 + "****".length)
      },
      changeCascader(val) { //改变部门函数
        let temp = ""
        val.map((item, index) => {
          temp += item + ","
        })
        this.addForm.personList = temp
      },
      gcodeFn(val) { // 部门码
        let temp = ""
        val.map((item, index) => {
          temp += item + ","
        })
        return temp
      },
      healthstatusFn(val) { //症状码
        let temp = ""
        val.map((item, index) => {
          temp += item + ","
        })
        return temp
      },
      addCommit() { //添加请求

        console.log("-------", this.addForm)
        if (this.addForm.image == undefined) {
          this.$message({
            message: "请先上传照片",
            type: 'error',
          })
          return
        }

        this.$http({
          url: this.$http.adornUrl('/tools/device/save'),
          method: 'POST',
          data: this.$http.adornData({
            name: this.addForm.name,
            status: this.addForm.status,
            fileurl: this.addForm.image.url,
            filename: this.addForm.image.name,

          })
        }).then((data) => {
          if (data.data.code == 0) {
            this.dialogFormVisible = false
            this.$message({
              message: "保存成功",
              type: 'success',
              onClose: () => {
                this.getDataList()
              }
            })
          } else {
            this.$message({
              message: data.data.msg,
              type: 'wraning',
            })
          }
        })
      },
      updateHandle(row) { //编辑
        this.dialogFormVisible = true
        this.title = "编辑人员信息"
        this.addForm.name = row.name;
        this.addForm.status = String(row.status);


        this.editData = row

        console.log("-----------------", row)
        // this.healthstatus = healthstatus.split(",")    
        console.log(process.env.imgurl)
        
        let tem = row.fileurl.replace(this.rep, process.env.imgurl) 
        console.log("图片地址", tem)
        this.addForm.url = tem;
        this.addForm.url = row.filename;
        this.fileList = [{
          name: row.filename,
          url: tem,
        }]
      },
      editCommit() { //编辑请求

        console.log("---122222222222----", this.addForm)
        if (this.fileList.length == 0) {
          this.$message({
            message: "请先上传照片",
            type: 'error',
          })
          return
        }
        
        this.$http({
          url: this.$http.adornUrl('tools/device/update'),
          method: 'POST',
          data: this.$http.adornData({
            name: this.addForm.name,
            status: this.addForm.status,
            id: this.editData.id,
            fileurl: this.addForm.url,
            filename: this.addForm.url,
          })
        }).then((data) => {
          if (data.data.code == 0) {
            if (data.data.code == 0) {
              this.dialogFormVisible = false
              this.$message({
                message: "更新成功",
                type: 'success',
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message({
                message: data.data.msg,
                type: 'wraning',
              })
            }
          }
        })
      },
      commit() { //提交

        if (Object.keys(this.editData).length == 0) { // 新增
          this.addCommit()
        } else { //修改
          this.editCommit()
        }
      },
      add() { // 添加
        this.dialogFormVisible = true
        this.title = "新增人员"
        this.editData = {}
        this.addForm = {}
        this.gcode = []
        this.healthstatus = []
        this.fileList = []
        // this.ishealthstatus = true
        this.FFFhealthstatus = ""
      },
      getDataList() { // 获取数据列表
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('tools/device/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'name': this.dataForm.name,
          })
        }).then(({
          data
        }) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
            this.rep = data.photoRep
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      sizeChangeHandle(val) { // 每页数
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      currentChangeHandle(val) { // 当前页
        this.pageIndex = val
        this.getDataList()
      },
      selectionChangeHandle(val) { // 多选
        this.dataListSelections = val
      },
      deleteHandle(id) { // 删除
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.roleId
        })
        this.$confirm(`确定删除`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('tools/device/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({
            data
          }) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      }
    }
  }

</script>

<style scoped>
  .groupgroup {
    margin-top: 4% !important;
    
  }

</style>
