<template>
    <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="商品名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="商品名称"></el-input>
      </el-form-item>
      <el-form-item label="价格" prop="price">
        <el-input v-model="dataForm.price" placeholder="价格"></el-input>
      </el-form-item>
      <el-form-item label="类型" prop="typeId">
        <el-input v-model="dataForm.selectName" placeholder="请点击选择一个类型" @focus="selectTree()"></el-input>
      </el-form-item>
      <el-dialog :visible.sync="treeVisible" :append-to-body="true" v-if="treeVisible">
        <el-form :model="form" ref="treeForm">
          <el-tree
          :data="treeList"
          :props="TreeProps"
          node-key="id"
          ref="listTree"
          @node-click="handleNodeClick">
          </el-tree>
        </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="callOf()">取 消</el-button>
        <el-button type="primary" @click="submitTree()">确 定</el-button>
      </div>
      </el-dialog>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary"  @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>
<script>
 import { treeDataTranslate } from '@/utils'
export default {
   data(){
       return {
            visible: false,
            treeVisible: false,
            formLabelWidth: '10px',
            treeList: [],
            TreeProps: {
                 label: 'name',
                 children: 'children'
            },
            dataForm: {
               id: 0,
               name: '',
               price: '',
               typeId:'',
               selectName: ''
           },
           form:{

           },
           dataRule:{
               name: [
                   {required: true, message: '商品名称不能为空', trigger: 'blur'}
               ],
               price: [
                   {required: true, message: '商品价格不能为空', trigger: 'blur'}
               ]
           }
       }
   },
   methods: {
       init(id){
           this.dataForm.id = id || 0
           this.$http({
               url: this.$http.adornUrl('/sys/shop/selectTree'),
               method: 'get',
               params: this.$http.adornParams()
           }).then(({data}) =>{
               this.treeList = data.data
           }).then(() => {
          this.visible = true
          this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
           // this.$refs.listTree.setCheckedKeys([])
          })
        }).then(()=>{
            if(this.dataForm.id){
              this.$http({
              url: this.$http.adornUrl(`/sys/shop/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.shop.name
                this.dataForm.price = data.shop.price
              }
            })
            }
        })
       },
       // 点击节点触发的点击事件
       handleNodeClick(data){
           //console.log("点击的节点id",data.id)
           this.dataForm.typeId = data.id
           this.dataForm.selectName = data.name
           //console.log("点击hou的节点id", this.dataForm.typeId)
       },
       // 表单提交
    dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/shop/${!this.dataForm.id ? 'add' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'price': this.dataForm.price,
                'typeId': this.dataForm.typeId
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList');
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      },
      selectTree(){
        this.treeVisible = true
      },
      submitTree(){
        this.treeVisible = false
        this.$refs['treeForm'].resetFields();
      },
      callOf(){
        this.treeVisible = false
        this.$refs['treeForm'].resetFields();
      }
   }
}
</script>

