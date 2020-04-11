<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="140px">
      <el-form-item label="品牌名" prop="name">
        <el-input v-model="dataForm.name" placeholder="品牌名"></el-input>
      </el-form-item>
      <el-form-item label="品牌logo地址" prop="logo">
        <!--<el-input v-model="dataForm.logo" placeholder="品牌logo地址"></el-input>-->
        <single-upload v-model="dataForm.logo"></single-upload>
      </el-form-item>
      <el-form-item label="介绍" prop="descript">
        <el-input v-model="dataForm.descript" placeholder="介绍"></el-input>
      </el-form-item>
      <el-form-item label="显示状态" prop="showStatus">
        <el-switch
          v-model="dataForm.showStatus"
          active-color="#13ce66"
          inactive-color="#ff4949">
        </el-switch>
      </el-form-item>
      <el-form-item label="检索首字母" prop="firstLetter">
        <el-input v-model="dataForm.firstLetter" placeholder="检索首字母"></el-input>
      </el-form-item>
      <el-form-item label="排序" prop="sort">
        <el-input v-model.number="dataForm.sort" placeholder="排序"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import SingleUpload from '../../../components/upload/singleUpload'

  export default {
    components: {SingleUpload},
    data () {
      return {
        visible: false,
        dataForm: {
          brandId: 0,
          name: '',
          logo: '',
          descript: '',
          showStatus: 1,
          firstLetter: '',
          sort: 0
        },
        dataRule: {
          name: [
            {required: true, message: '品牌名不能为空', trigger: 'blur'}
          ],
          logo: [
            {required: true, message: '品牌logo地址不能为空', trigger: 'blur'}
          ],
          descript: [
            {required: true, message: '介绍不能为空', trigger: 'blur'}
          ],
          showStatus: [
            {required: true, message: '显示状态不能为空', trigger: 'blur'}
          ],
          firstLetter: [
            {
              validator: (rule, value, callback) => {
                if (!/^[a-zA-Z]$/.test(value)) {
                  callback(new Error('请输入任意字母'))
                } else if (value === '') {
                  callback(new Error('首字母为必填项'))
                } else {
                  callback()
                }
              }, trigger: 'blur'
            }
          ],
          sort: [
            { // 自定义校验规则
              validator: (rule, value, callback) => {
                if (!Number.isInteger(value) || value < 0) {
                  callback(new Error('请输入大于0的整数'))
                } else if (value === '') {
                  callback(new Error('排序字段为必填项'))
                } else {
                  callback()
                }
              }, trigger: 'blur'
            }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.brandId = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.brandId) {
            this.$http({
              url: this.$http.adornUrl(`/product/brand/info/${this.dataForm.brandId}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                let res = data.data
                this.dataForm.name = res.name
                this.dataForm.logo = res.logo
                this.dataForm.descript = res.descript
                this.dataForm.showStatus = res.showStatus
                this.dataForm.firstLetter = res.firstLetter
                this.dataForm.sort = res.sort
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
              url: this.$http.adornUrl(`/product/brand/${!this.dataForm.brandId ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'brandId': this.dataForm.brandId || undefined,
                'name': this.dataForm.name,
                'logo': this.dataForm.logo,
                'descript': this.dataForm.descript,
                'showStatus': this.dataForm.showStatus ? 1 : 0,
                'firstLetter': this.dataForm.firstLetter,
                'sort': this.dataForm.sort
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
      }
    }
  }
</script>
