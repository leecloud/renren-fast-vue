<template>
    <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="部门名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="名称"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="callOf()">取消</el-button>
      <el-button type="primary"  @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>
<script>
export default {
   data(){
       return {
            visible: false,
            dataForm: {
               id: 0,
               name: ''
           },
           dataRule:{
               name: [
                   {required: true, message: '商品名称不能为空', trigger: 'blur'}
               ]
           }
       }
   },
   methods: {
       init(id){
           this.dataForm.id = id || 0
           this.visible = true
           if(this.dataForm.id){
              this.$http({
              url: this.$http.adornUrl(`/sys/dep/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.data.name
              }
            })
            }
       },
       // 表单提交
    dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/dep/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.$refs['dataForm'].rest
                    this.visible = false
                    this.$emit('refreshDataList');
                    this.$nextTick(() => {
                    this.$refs['dataForm'].resetFields()
                    // this.$refs.listTree.setCheckedKeys([])
                    })
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      },
      // 取消
      callOf(){
          this.visible = false
          this.$nextTick(()=>{
              this.$refs['dataForm'].resetFields()
          })
      }
   }
}
</script>

