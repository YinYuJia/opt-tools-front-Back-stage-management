<template>
  <el-dialog 
  width="80%" 
  :visible.sync="visible" 
  title="需求信息详细" 
  top="5vh" 
  @close="back" 
  :close-on-click-modal="false" z-index="10000">
    <el-tabs v-model="activeName">
      <el-tab-pane label="基本信息" name="info">
        <detail-info ref="detailInfo"></detail-info>
      </el-tab-pane>
      <el-tab-pane label="交流记录" name="record">
        <detail-record ref="detailRecord"></detail-record>
      </el-tab-pane>
      <el-tab-pane label="@事项" name="matter">
        <detail-assign ref="detailAssign"></detail-assign>
      </el-tab-pane>
      <el-tab-pane label="我的待办" name="myMatter">
        <detail-deal ref="detailDeal"></detail-deal>
      </el-tab-pane>
      <el-tab-pane label="附件上传" name="file">
        <detail-file ref="detailFile"></detail-file>
      </el-tab-pane>
    </el-tabs>
  </el-dialog>
</template>
<script>
  import detailInfo from './detail-info'
  import detailRecord from './detail-record'
  import detailAssign from './detail-assign'
  import detailDeal from './detail-deal'
  import detailFile from './detail-file'
  export default {
    components:{
      detailInfo,
      detailRecord,
      detailAssign,
      detailDeal,
      detailFile
    },
    data() {
      return {
        visible:false,
        activeName:''
      }
    },
    mounted() {
    },
    created() {

    },
    methods: {
      init(id){
        this.activeName = 'info'
        this.visible = true
        this.$nextTick(() => {
          this.$refs.detailInfo.init(id)
          this.$refs.detailRecord.init(id)
          this.$refs.detailAssign.init(id)
          this.$refs.detailDeal.init(id)
          this.$refs.detailFile.init(id)
        })
      },
      back() {
        this.$parent.detailViewChange(false)
        // this.detailVisible = false
      }
    }
  }
</script>
<style>
  #detailForm .el-form-item {
    margin-bottom: 5px
  }

  .upload-demo {
    width: 100%
  }

  .infinite-list {
    height: 200px;
    padding: 0;
    list-style: none;
    overflow: auto;
    width: 100%;
    margin: 0;
  }

  .infinite-list .infinite-list-item {
    margin: 5px 5px 5px 0px;
    line-height: 20px;
    font-size: 15px;
  }
</style>
