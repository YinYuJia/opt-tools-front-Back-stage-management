<template>
  <!-- 工具管理 -->
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
      <el-table-column prop="sbmc" header-align="center" align="center" width="180" label="设备名称">
      </el-table-column>
      <el-table-column prop="sfst" header-align="center" align="center" :formatter="sexFormatter" width="180"
        label="是否双套">
      </el-table-column>
      <el-table-column prop="ggxh" header-align="center" align="center" width="180" label="规格型号">
      </el-table-column>
      <el-table-column prop="sccj" header-align="center" align="center" width="180" label="生产厂家">
      </el-table-column>
      <el-table-column prop="ccdate" header-align="center" align="center" min-width="180" label="出厂日期">
      </el-table-column>
      <el-table-column prop="syqx" header-align="center" align="center" width="180" label="使用期限">
      </el-table-column>
      <el-table-column prop="ccbh" header-align="center" align="center" width="180" label="出厂编号">
      </el-table-column>
      <el-table-column prop="syzq" header-align="center" align="center" width="180" label="试验周期">
      </el-table-column>
      <el-table-column prop="bcsydate" header-align="center" align="center" width="180" label="本次试验">
      </el-table-column>
      <el-table-column prop="xcsydate" header-align="center" align="center" width="180" label="下次实验">
      </el-table-column>
      <el-table-column prop="bz" header-align="center" align="center" width="180" label="备注">
      </el-table-column>
      <el-table-column prop="creatdate" header-align="center" align="center" width="180" label="创建时间">
      </el-table-column>
      <el-table-column prop="creatdate" header-align="center" align="center" width="280" label="操作时间">
      </el-table-column>
      <el-table-column fixed="right" header-align="center" align="center" width="250" label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('sys:role:update')" type="text" size="small" @click="updateHandle(scope.row)">修改
          </el-button>
          <span>|</span>
          <el-button v-if="isAuth('sys:role:delete')" style="color:red" type="text" size="small"
            @click="deleteHandle(scope.row.id)">删除</el-button>
          <span>|</span>
          <el-button v-if="isAuth('sys:role:delete')" style="color:red" type="text" size="small"
            @click="upLoadImg(scope.row)">上传图纸</el-button>
          <span>|</span>
          <el-button v-if="isAuth('sys:role:delete')" style="color:red" type="text" size="small"
            @click="searchUpLoadImg(scope.row)">查询图纸</el-button>
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
        <el-form-item label="设备名称" :label-width="formLabelWidth">
          <el-input style="width:90%" v-model="addForm.sbmc" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="设备类" :label-width="formLabelWidth">
          <el-select style="width:90%" v-model="addForm.deviceid" placeholder="请选择设备类">
            <el-option v-for="item in deviceidList" :key="item.id" :label="item.name" :value="item.id"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="规格型号" :label-width="formLabelWidth">
          <el-input style="width:90%" v-model="addForm.ggxh" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="生产厂家" :label-width="formLabelWidth">

          <el-input style="width:90%" v-model="addForm.sccj" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="出厂日期" :label-width="formLabelWidth">
          <el-date-picker style="width:90%" v-model="addForm.ccdate" format="yyyy-MM-dd" value-format="yyyy-MM-dd"
            type="date" placeholder="选择日期时间">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="使用期限" :label-width="formLabelWidth">
          <el-input style="width:90%" v-model="addForm.syqx" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="出厂编号" :label-width="formLabelWidth">
          <el-input style="width:90%" v-model="addForm.ccbh" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="是否双套" :label-width="formLabelWidth">
          <el-radio-group v-model="addForm.sfst">
            <el-radio label="0">是</el-radio>
            <el-radio label="1">不是</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="实验周期" :label-width="formLabelWidth">
          <el-input style="width:90%" v-model="addForm.syzq" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="本次实验" :label-width="formLabelWidth">
          <el-date-picker style="width:90%" v-model="addForm.bcsydate" format="yyyy-MM-dd" value-format="yyyy-MM-dd"
            type="date" placeholder="选择日期时间">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="下次实验" :label-width="formLabelWidth">
          <el-date-picker style="width:90%" v-model="addForm.xcsydate" format="yyyy-MM-dd" value-format="yyyy-MM-dd"
            type="date" placeholder="选择日期时间">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="备注" :label-width="formLabelWidth">
          <el-input style="width:90%" v-model="addForm.bz" autocomplete="off"></el-input>
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel()">取 消</el-button>
        <el-button type="primary" @click="commit()">确 定</el-button>
      </div>
    </el-dialog>

    <el-dialog title="上传图纸" :visible.sync="uploadfile">
      <el-form :model="addForm">
        <el-form-item label-width="0">
          <el-upload  action="#" list-type="picture-card" :on-preview="handlePictureCardPreview"
            :on-remove="handleRemove" :before-remove="beforeRemove" :http-request="uploadAvatar" :before-upload="beforeAvatarUpload"
            :file-list="fileList1">
            <i class="el-icon-plus"></i>
          </el-upload>
          <el-dialog :visible.sync="dialogVisible">
            <img width="100%" :src="dialogImageUrl" alt="">
          </el-dialog>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel()">取 消</el-button>
        <el-button type="primary" @click="commit()">确 定</el-button>
      </div>
    </el-dialog>
    <el-dialog title="照片查看" :visible.sync="dialogimg">
      <img style="width:60%;height:670px;margin-left:50%;transform: translate(-50%);" :src="imgurl" alt="">
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        imgurl: "",
        dialogimg: false,
        uploadfile: false,
        uploadfileData: {},
        fileList1:[],
        url: "",
        title: "新增人员",
        FFFhealthstatus: "",
        selectDisabled:false,
        multipleSelection: [],
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
        deviceidList: [],
        formLabelWidth: '180px',
        dataForm: {
          name: '',
        },
        gcode: [],
        addForm: {
          sbmc: "",

          sfst: "",

          image: ""
        },
        healthstatus: [],
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],

        isId: "",
        editData: {},
        fileList: [],
        rep: "",
        dialogImageUrl: '',
        dialogVisible: false,
        picList: [],
        photoRep: "",
        templist: [],
      }
    },
    watch: {

    },
    activated() {
      this.getDataList()
      this.getdeviceidList()
    },
    methods: {
      getdeviceidList() {
        this.$http({
          url: this.$http.adornUrl('tools/device/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': 1,
            'limit': 1000000,
            'name': "",
          })
        }).then(({
          data
        }) => {
          if (data && data.code === 0) {
            this.deviceidList = data.page.list
          } else {}
        })
      },
      searchUpLoadImg(row) {
        console.log(row)
        this.$http({
          url: this.$http.adornUrl('tools/file/queryAllList'),
          method: 'get',
          params: this.$http.adornParams({
            'projectid': row.id,
          })
        }).then(({
          data
        }) => {
          if (data && data.code === 0) {
            this.photoRep = data.photoRep
            let temp = []
            data.list.map((item, index) => {
              item.url = item.url.replace(this.photoRep, process.env.imgurl)
              temp.push({
                name:item.name,
                url:item.url,
                id:item.id
              })
            })
            this.fileList1 = temp
            console.log("filelist1---",this.fileList1)
            this.uploadfile = true;
            this.selectDisabled = true
            console.log(data.list)
            // [{name: 'food.jpg', url: 'https://xxx.cdn.com/xxx.jpg'}]
            
          } else {}

        })
      },
      upLoadImg(row) {
        this.uploadfile = true
        this.uploadfileData = row
        this.selectDisabled = false;
      },
      look(val) {
        // console.log("]]]]",val.fileurl)
        let tem = val.fileurl.replace(this.rep, process.env.imgurl)
        console.log(tem)
        this.imgurl = tem
        this.dialogimg = true
      },
      changeUpload(val) {
        console.log("val--", val)
        //  this.addForm.image.url
      },

      uploadAvatar(item) {
        console.log("[[[[[[[", this.uploadfileData)
        const formData = new FormData()
        formData.append('file', item.file)
        const uid = item.file.uid
        console.log(item.file)
        let myform = new FormData()

        myform.append('file', item.file);
        myform.append('projectid', this.uploadfileData.id);
        console.log("开始请求")
        this.$http({
          url: this.$http.adornUrl('tools/file/uploadFile'),
          method: 'POST',
          data: myform
        }).then((data) => {
          console.log("返回值-----", data)
          if (data.data.code == 0) {
            this.$message({
              message: "上传成功",
              type: 'success'
            });
          } else {
            this.$message({
              message: "上传失败",
              type: 'warning'
            });
          }
        })
      },
      beforeAvatarUpload(file) {
        const isJPG = file.type === 'image/jpeg'
        const isPng = file.type === 'image/png'
        const isLt2M = file.size / 1024 / 1024 < 5

        if (!isJPG && !isPng) {
          this.$message.error('上传图片只能是 JPG或png 格式!')
        }
        if (!isLt2M) {
          this.$message.error('上传图片大小不能超过 5MB!')
        }
        return (isJPG || isPng) && isLt2M
      },
      beforeRemove(file,filelist) {
         console.log("22222",file)
         console.log("22222",filelist)
         return true
      },
      handleRemove(file, fileList) {
        console.log("file000000",file.id)
        this.$http({
          url: this.$http.adornUrl('tools/file/deleteFile'),
          method: 'POST',
          data: this.$http.adornData({
            fileId:file.id
          })
        }).then((data) => {
          if (data.data.code == 0) {
            
            this.$message({
              message: "删除成功",
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
              
        // this.fileList1 = fileList
        for (const i in this.picList) {
          if (this.picList[i].key === file.uid) {
            this.picList.splice(i, 1)
          }
        }
      },
      handlePictureCardPreview(file) {
        this.dialogImageUrl = file.url
        this.dialogVisible = true
      },
      /**
       * 清空上传组件
       */
      emptyUpload() {
        const mainImg = this.$refs.upload
        if (mainImg) {
          if (mainImg.length) {
            mainImg.forEach(item => {
              item.clearFiles()
            })
          } else {
            this.$refs.upload.clearFiles()
          }
        }
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
      cancel() {
        this.dialogFormVisible = false
        this.addForm = {}
      },
      sexFormatter(val) {
        if (val.sfst == 0) {
          return "是"
        } else {
          return "不是"
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
        this.$http({
          url: this.$http.adornUrl('/tools/deviceinfo/save'),
          method: 'POST',
          data: this.$http.adornData(this.addForm)
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
        this.addForm = row
        this.addForm.sfst = String(row.sfst)

        this.editData = row

        console.log("-----------------", this.addForm)
        // this.healthstatus = healthstatus.split(",")
        // this.addForm.image.fileurl = row.fileurl
        // this.addForm.image.filename = row.filename
        // let tem = row.fileurl.replace(this.rep, "http://192.168.0.185:8888") 
        // console.log("图片地址", tem)
        this.fileList = [{
          name: row.filename,
          url: row.fileurl,
        }]
      },
      editCommit() { //编辑请求

        console.log("---1修改22----", this.addForm)

        this.$http({
          url: this.$http.adornUrl('tools/deviceinfo/update'),
          method: 'POST',
          data: this.$http.adornData(this.addForm)
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
        this.FFFhealthstatus = ""
      },
      getDataList() { // 获取数据列表
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('tools/deviceinfo/list'),
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
            url: this.$http.adornUrl('tools/deviceinfo/delete'),
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
