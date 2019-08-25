<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="用户名" prop="account">
        <el-input v-model="dataForm.account" placeholder="登录帐号"></el-input>
      </el-form-item>
      <el-form-item label="旧密码" prop="old_pwd" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.old_pwd" type="text" placeholder="旧密码"></el-input>
      </el-form-item>
      <el-form-item label="新密码" prop="new_pwd" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.new_pwd" type="text" placeholder="新密码"></el-input>
      </el-form-item>
      <el-form-item label="确认密码" prop="confirm_pwd" :class="{ 'is-required': !dataForm.id }">
        <el-input v-model="dataForm.confirm_pwd" type="password" placeholder="确认密码"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import { isEmail, isMobile } from '@/utils/validate'
  export default {
    data () {
      var validatePassword = (rule, value, callback) => {
        if ( !/\S/.test(value)) {
          callback(new Error('密码不能为空'))
        } else {
          callback()
        }
      }
      var validateComfirmPassword = (rule, value, callback) => {
        if ( !/\S/.test(value)) {
          callback(new Error('确认密码不能为空'))
        } else if (this.dataForm.new_pwd !== value) {
          callback(new Error('确认密码与密码输入不一致'))
        } else {
          callback()
        }
      }
      return {
        visible: false,
        roleList: [],
        dataForm: {
          id: 0,
          account: '',
          old_pwd: '',
          new_pwd: '',
          confirm_pwd: '',
        },
        dataRule: {
          account: [
            { required: true, message: '用户名不能为空', trigger: 'blur' }
          ],
          old_pwd: [
            { validator: validatePassword, trigger: 'blur' }
          ],
          new_pwd: [
            { validator: validatePassword, trigger: 'blur' }
          ],
          confirm_pwd: [
            { validator: validateComfirmPassword, trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.$http({
          url: this.$http.adornUrl('/sys/role/select'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.roleList = data && data.code === 0 ? data.list : []
        }).then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
          })
        }).then(() => {
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/sys/user/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.userName = data.user.username
                this.dataForm.salt = data.user.salt
                this.dataForm.email = data.user.email
                this.dataForm.mobile = data.user.mobile
                this.dataForm.roleIdList = data.user.roleIdList
                this.dataForm.status = data.user.status
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
              url: this.$http.adornUrl('v1/change'),
              method: 'put',
              data: this.$http.adornData({
                'userId': this.dataForm.id || undefined,
                'account': this.dataForm.account,
                'old_pwd': this.dataForm.old_pwd,
                'new_pwd': this.dataForm.new_pwd,
                'confirm_pwd': this.dataForm.confirm_pwd
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '修改密码成功',
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
