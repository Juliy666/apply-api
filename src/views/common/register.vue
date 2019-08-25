<template>
  <el-dialog
    :title="!dataForm.id ? '注册' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="100px">
      <el-form-item label="账号(4-12位)" prop="account">
        <el-input v-model="dataForm.account" placeholder="账号(4-12位)"></el-input>
      </el-form-item>
      <el-form-item label="密码(6-18位)" prop="password">
        <el-input v-model="dataForm.password" placeholder="密码(6-18位)"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">注册</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          account: '',
          password: '',
        },
        dataRule: {
          account: [
            { required: true, message: '用户名不能为空', trigger: 'blur' }
          ],
          password: [
            { required: true, message: '密码不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              method: 'get',
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.account = data.register.account
                this.dataForm.password = data.register.password
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
              url: this.$http.adornUrl('v1/register/社团编号'),
              method: 'post',
              data: this.$http.adornData({
                /* 'jobId': this.dataForm.id || undefined, */
                'account': this.dataForm.account,
                'password': this.dataForm.password
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '注册成功',
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
      }
    }
  }
</script>
