<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"

    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="考卷名称" prop="paperName">
        <el-input v-model="dataForm.paperName" placeholder="考卷名称"></el-input>
      </el-form-item>
      <el-form-item label="序号" prop="subjectNo">
        <el-input v-model="dataForm.subjectNo" placeholder="考卷序号 例如8就是第八题"></el-input>
      </el-form-item>
      <el-form-item label="试题描述" prop="subjectDesc">
        <el-input v-model="dataForm.subjectDesc" placeholder="试题描述"></el-input>
      </el-form-item>
      <el-form-item label="考题图片" prop="subjectPic">
       <!--<el-input v-model="dataForm.subjectPic" placeholder="考题图片" value=""></el-input>-->
        <upload v-if="uploadVisible" ref="upload" @showbox="getPicUrl"></upload>
        <el-button type="text"  @click="uploadHandle()" >上传文件</el-button>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>

</template>

<script>
  import Upload from './upload'
  export default {
    data () {
      return {
        visible: false,
        uploadVisible: true,
        dataForm: {
          paperId: 0,
          paperName: '',
          subjectNo: '',
          subjectDesc: '',
          subjectPic: '',
          createDate: '',
          tsrefreshtime: ''
        },
        dataRule: {
          paperName: [
            { required: true, message: '考卷名称不能为空', trigger: 'blur' }
          ],
          subjectNo: [
            { required: true, message: '考卷序号 例如 8就是第八题', trigger: 'blur' }
          ],
          subjectDesc: [
            { required: false, message: '考卷描述不能为空', trigger: 'blur' }
          ],
          subjectPic: [
            { required: false, message: '考题图片不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    components: {
      Upload
    },
    activated () {
      this.getPicUrl()
    },
    methods: {
      init (id) {
        this.dataForm.paperId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.paperId) {
            this.$http({
              url: this.$http.adornUrl(`/generator/testpaper/info/${this.dataForm.paperId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.paperName = data.testpaper.paperName
                this.dataForm.subjectNo = data.testpaper.subjectNo
                this.dataForm.subjectDesc = data.testpaper.subjectDesc
                this.dataForm.subjectPic = data.testpaper.subjectPic
                this.dataForm.createDate = data.testpaper.createDate
                this.dataForm.tsrefreshtime = data.testpaper.tsrefreshtime
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/testpaper/${!this.dataForm.paperId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'paperId': this.dataForm.paperId || undefined,
                'paperName': this.dataForm.paperName,
                'subjectNo': this.dataForm.subjectNo,
                'subjectDesc': this.dataForm.subjectDesc,
                'subjectPic': this.dataForm.subjectPic
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      },
      // 上传文件
      uploadHandle () {
        this.uploadVisible = true
        this.$nextTick(() => {
          this.$refs.upload.init()
        })
      },
      getPicUrl (msg) {
        this.dataForm.subjectPic = msg
      }

    }
  }
</script>
