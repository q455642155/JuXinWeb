<template>

  <el-dialog custom-class="dia-class" :visible.sync="visibleP" :before-close="handleClose" center="">
    <header slot="title">
      <span class="title">
        {{getTitle}}
      </span>
    </header>
    <section class="layout loan-contract-form">
      <el-form ref="form" :model="detailsP" :rules="rules" status-icon label-width="130px">
        <el-row>
          <el-col :span="11" class="flex">
            <el-form-item label="实放金额:" prop="loanAmt">
              <el-input v-model="detailsP.loanAmt">
                <template slot="append">元</template>
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="11" class="flex">
            <el-form-item label="还款日期: " prop="repayDate">
              <span>{{detailsP.repayDate | dateFormat}}</span>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col>
            <el-form-item label="合同上传:">
              <form ref="uploadForm" method="post" name="fileInfo" enctype="multipart/form-data">
                <input ref="selectFile" type="file" name="contractUploadFile" @change="selectFile">
                <el-button type="primary" size="mini" @click="uploadFile">上传</el-button>
              </form>
              <!-- <el-upload class="upload-demo" ref="upload" :action="uploadUrl" :auto-upload="false" :on-success="handleSuccess">
                <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
                <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传</el-button>
              </el-upload> -->
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
            <el-col>
              <table border="1" bordercolor="#931719" style="width:100%;border-collapse:collapse;text-align:center;">
                  <caption style="height:30px;">已上传列表</caption>
                  <thead>
                    <tr style="height:40px;">
                      <th>序号</th>
                      <th>附件名称</th>
                      <th>操作</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr style="height:40px;" v-for="(item, index) in this.contractList" :key="index">
                      <td style="width:80px;">{{index + 1}}</td>
                      <td><a :href="item.contractUploadFileUrl" style="display:block;" target="_blank">{{item.contractUploadFileName}}</a></td>
                      <td style="width:120px;"><el-button type="danger" size="mini" @click="deleteFile(index)">删除</el-button></td>
                    </tr>
                  </tbody>
              </table>
            </el-col>
        </el-row>
      </el-form>
    </section>
    <footer slot="footer" :style="'clear:both'">
      <el-button type="primary" @click="uploadContract">确定</el-button>
      <el-button type="default" @click="handleClose">取消</el-button>
    </footer>
  </el-dialog>
</template>
<style lang="scss">
.layout.loan-contract-form {
  margin-top: 10px;
  .el-row {
    margin-top: 10px;
  }
}

.layout.loan-contract-form .flex {
  display: flex;
  label {
    width: 150px;
    height: 40px;
    line-height: 40px;
  }
  span {
    height: 40px;
    line-height: 40px;
  }
  .el-select,
  .el-date-editor.el-input,
  .el-date-editor.el-input__inner {
    width: 180px;
  }
  .el-input-group__append {
    padding: 0 5px;
  }
}
</style>

<script>
import DialogClose from '@/mixins/suplier/Ar/DialogClose'
import Common from '@/mixins/common'
// import { checkNumberPire } from '@/util/validate' // 校验数字
import { debounce } from '@/util/util' // 防抖函数
import { loadingConf } from '@/config/common' // 获取加载配置
// import { apiUrl } from '@/config/env.js'
import { mapGetters } from 'vuex'

export default {
  props: ['visibleP', 'detailsP'],
  mixins: [DialogClose, Common],
  data () {
    return {
      // uploadUrl: apiUrl + '/creditLoan/creditLoanUploadFile.do', // 文件上传地址
      fileInfo: '', // 文件信息
      loanAmt: '',
      // 校验规则
      rules: {
        loanAmt: [
          { required: true, message: '请输入实放金额', trigger: 'blur' }
        ]
      }
    }
  },
  computed: {
    getTitle () {
      return this.detailsP.loanId + '合同确认'
    },
    ...mapGetters(['token']),
    // 附件列表
    contractList () {
      // return this.uniqueData(this.detailsP.contractUploadFileList)
      return this.detailsP.contractUploadFileList
    }
  },
  methods: {
    // 生成合同
    uploadContract: debounce(submit, 1000, true),
    // 去重数据
    // uniqueData (list) {
    //   var newArr = [this.detailsP.contractUploadFileList[0]]
    //   for (var i = 1; i < list.length; i++) {
    //     var listItem = list[i]
    //     var repeat = false
    //     for (var j = 0; j < newArr.length; j++) {
    //       if (listItem.contractUploadFileName === newArr[j].contractUploadFileName) {
    //         repeat = true
    //         break
    //       }
    //     }
    //     if (!repeat) {
    //       newArr.push(listItem)
    //     }
    //   }
    //   return newArr
    // },
    // 选择文件
    selectFile (e) {
      this.fileInfo = e.target.files[0]
    },
    // 上传文件
    uploadFile () {
      var formData = new FormData(this.$refs.uploadForm)
      formData.append('contractUploadFile', this.fileInfo)
      formData.append('loanId', this.detailsP.loanId)
      formData.append('contractUploadFileName', this.fileInfo.name)
      if (this.fileInfo.length === 0 || this.$refs.selectFile.value === '' || this.$refs.selectFile.value === null) {
        this.$message({
          type: 'error',
          message: '请选择文件之后再上传'
        })
      } else {
        // 显示加载动画
        // const loading = this.$loading(loadingConf.get())
        this.axios.post('/factoringCreditLoan/creditLoanManualContractUploadFile.do', formData).then(res => {
          if (res.data.status) {
            // loading.close()
            var contractListInfo = {
              'contractUploadFileUrl': res.data.data.contractUploadFileUrl,
              'contractUploadFileName': res.data.data.contractUploadFileName
            }
            this.contractList.push(contractListInfo)
            this.$refs.selectFile.value = ''
            this.$parent.fresh()
          } else {
            this.$message.error(res.data.msg)
          }
        }).catch(err => {
          console.log(err)
          // erroShow.call(this, err, loading)
        })
      }
    },
    // 删除文件
    deleteFile (index) {
      this.$confirm(`您确定要删除吗?`, `提示`, {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
        center: true
      }).then(() => {
        this.axios.post('/factoringCreditLoan/creditLoanManualContractDeleteFile.do', { loanId: this.detailsP.loanId, contractUploadFileUrl: this.contractList[index].contractUploadFileUrl }).then(res => {
          if (res.data.status) {
            this.$message({
              type: 'success',
              message: res.data.data
            })
            this.contractList.splice(index, 1)
            this.$parent.fresh()
          } else {
            this.$message.error(res.data.msg)
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '操作已取消'
        })
      })
    }
  }
}
// 合同文件生成
function submit () {
  // 表单验证
  this.$refs.form.validate((valid) => {
    if (valid) {
      // 组合数据
      const param = {
        loanId: this.detailsP.loanId, // 融资编号Id
        applyAmt: this.detailsP.applyAmt, // 实放金额
        repayDate: this.detailsP.repayDate, // 还款日期
        contractUploadFileList: this.contractList // 合同列表
      }
      // 判断table中有没有附件
      if (this.contractList.length === 0) {
        this.$message({
          type: 'info',
          message: '列表中没有合同附件，请上传附件'
        })
      } else {
        // 显示加载图标
        const loading = this.$loading(loadingConf.get())
        this.axios.post('/factoringCreditLoan/generateManualContract.do', param).then(res => {
          let type = res.data.status ? 'success' : 'error'
          this.$message({
            showClose: true,
            message: res.data.data ? res.data.data : '返回结果错误，请联系管理员',
            type: type
          })
          if (res.data.status) {
            loading.close()
            this.$parent.fresh()
            this.contractList = []
            this.handleClose()
          }
        }).catch(err => {
          console.log(err)
          // erroShow.call(this, err, loading)
        })
      }
    }
  })
}
</script>
