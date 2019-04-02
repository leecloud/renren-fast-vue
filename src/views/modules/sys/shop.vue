<template>
<el-row :gutter="20">
    <!--tree树开始-->
  <el-col :span="4">
      <div class="grid-content bg-purple">
          <el-button type="primary" plain @click="addRoot()">添加根节点</el-button>
          <el-tree 
          :data="treeList" 
          :props="defaultProps" 
          @node-click="handleNodeClick"
          :render-content="renderContent"></el-tree>
      </div>
  </el-col>
  <el-col :span="20">
      <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
        <el-form-item>
            <el-input v-model="dataForm.name" placeholder="商品名称" clearable></el-input>
        </el-form-item>
        <el-form-item>
            <el-button @click="getDataList()">查询</el-button>
            <el-button v-if="isAuth('sys:user:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
            <el-button v-if="isAuth('sys:user:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
        </el-form-item>
    </el-form>
      <el-table
        :data="shopList"
        border
        v-loading="dataListLoading"
        @selection-change="selectionChangeHandle"
        style="width: 100%;">
        <el-table-column
            type="selection"
            header-align="center"
            align="center"
            width="50">
        </el-table-column>
        <el-table-column
            prop="id"
            header-align="center"
            align="center"
            width="80"
            label="ID">
        </el-table-column>
        <el-table-column
            prop="name"
            header-align="center"
            align="center"
            label="商品名称">
        </el-table-column>
        <el-table-column
            prop="price"
            header-align="center"
            align="center"
            label="价格">
        </el-table-column>
        <el-table-column
            prop="typeName"
            header-align="center"
            align="center"
            label="类型">
        </el-table-column>
        <el-table-column
            prop="createUserName"
            header-align="center"
            align="center"
            label="创建人">
        </el-table-column>
        <el-table-column
            prop="createTime"
            header-align="center"
            align="center"
            width="180"
            label="创建时间">
        </el-table-column>
        <el-table-column
            prop="updateTime"
            header-align="center"
            align="center"
            width="180"
            label="修改时间">
        </el-table-column>
        <el-table-column
            prop="delFlag"
            header-align="center"
            align="center"
            label="是否删除">
            <template slot-scope="scope">
            <el-tag v-if="scope.row.delFlag === false" size="small" type="danger">否</el-tag>
            <el-tag v-else size="small">是</el-tag>
            </template>
        </el-table-column>
        <el-table-column
            fixed="right"
            header-align="center"
            align="center"
            width="150"
            label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('sys:user:update')" type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button v-if="isAuth('sys:user:delete')" type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
        </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      @selection-change="selectionChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
     <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </el-col>
  <el-dialog title="编辑节点名称" :visible.sync="dialogFormVisible" v-if="dialogFormVisible">
    <el-form :model="form" ref="refname">
      <el-form-item label="节点名称" :label-width="formLabelWidth" prop="name">
        <el-input v-model="form.name" ></el-input>
      </el-form-item>
    </el-form>
  <div slot="footer" class="dialog-footer">
    <el-button @click="callOf('refname')">取 消</el-button>
    <el-button type="primary" @click="submit(form)">确 定</el-button>
  </div>
</el-dialog>
</el-row>

  
</template>
<script>
 import AddOrUpdate from './shop-add-or-update'
export default {
  data(){
     return{
          dataForm: {
          name: ''
        },
         treeList: [],
         defaultProps: {
          children: 'children',
          label: 'name'
        },
        shopList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        form: {
          id: '',
          pid: '',
          name: ''
        },
        dialogFormVisible: false,
        formLabelWidth: '100px',
        current: ''
        
     }
     
  },
   // 引入的组件
    components: {
      AddOrUpdate
    },
    
  // 一进这个页面就会执行getList这个方法
    
    mounted:function(){
      this.getList();
      this.getDataList()
    },
   methods: {
       // 初始化tree树
       getList(){
           this.$http({
               url: this.$http.adornUrl('/sys/shop/selectTree'),
                method: 'get'
           }).then(({data}) => {
          if (data && data.code === 0) {
            this.treeList = data.data
            // console.log(data.data)
          } else {
            this.treeList = []
          }
        })
       },
       //加载所有数据分页
        getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/sys/shop/page'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'name': this.dataForm.name
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            //console.log(data.data)
            this.shopList = data.data.list
            this.totalPage = data.data.totalCount
          } else {
            this.shopList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
     
       // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.shopList()
      },
    // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        // 点击下一页
        this.getDataList()
      },
       // 多选
    selectionChangeHandle (val) {
        this.dataListSelections = val
      },
       // 新增 / 修改
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
       // 点击触发的事件根据点击的id去查询对应的商品信息
    handleNodeClick(data) {
        this.dataListLoading = true
       // console.log("点击的id"+data.id);
        this.$http({
           // url: this.adornUrl("/sys/shop/selectByTypeId/"+_this.data.id),
            url: this.$http.adornUrl(`/sys/shop/selectByTypeId/${data.id}`),
            method: 'get',
            params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'name': this.dataForm.name
          })
        }).then(({data})=>{
            if(data && data.code === 0){
               // console.log(".......",data)
                this.shopList = []
                this.shopList = data.data.list
                this.totalPage = data.data.totalCount
            }else{
                this.shopList = []
                this.totalPage = 0
            }
             this.dataListLoading = false
        })
      },
    //    // 删除
      deleteHandle (id) {
        var shopIds = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${shopIds.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/sys/shop/delete'),
            method: 'post',
            data: this.$http.adornData(shopIds, false)
          }).then(({data}) => {
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
        }).catch(() => {})
      },
      //添加根节点
    addRoot(){
        this.$http({
           // url: this.adornUrl("/sys/shop/selectByTypeId/"+_this.data.id),
            url: this.$http.adornUrl(`/sys/shop/addRoot`),
            method: 'post',
            data: this.$http.adornData({
            'name': "新增根节点"
          })
        }).then(({data})=>{
            if(data && data.code === 0){
              this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                    this.getList()
                  }
                })
            }else{
               this.$message.error(data.msg)
            }
        })
    },
    // 添加节点
    menuAdd(node,data,store){
      // 点击添加时打开dialog表单
      this.dialogFormVisible = true
     // this.current = data.id
      this.form.pid = data.id
     // alert(data.id)

    },
    submit(form){
      console.log(form.id)
        this.$http({
           // url: this.adornUrl("/sys/shop/selectByTypeId/"+_this.data.id),
            url: this.$http.adornUrl(`/sys/shop/${!this.form.id ? 'addTree' : 'updateTree'}`),
            method: 'post',
            data: this.$http.adornData({
            'id': this.form.id || undefined,
            'pid': this.form.pid,
            'name': this.form.name
          })
        }).then(({data})=>{
            if(data && data.code === 0){
              this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                    // 关闭dialog并清除缓存
                    //this.$refs['form'].resetFields()
                    this.$refs['refname'].resetFields();
                    this.dialogFormVisible = false
                    this.dataForm.id = ''
                    this.getList()
                  }
                })
            }else{
               this.$message.error(data.msg)
            }
        }) 
    },
    menuRemove(node,data,store){
      if(data.pid === 0){
        this.$message({
          message: '根节点不允许删除',
          type: 'warning'
        });
      }else{
       this.$http({
          url: this.$http.adornUrl(`/sys/shop/deleteType/${data.id}`),
          method: 'get',
          params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
      }
    },
    // 编辑节点，点击的时候打开表单并查询返回数据进行渲染
    menuEdit(node,data,store){
      this.dialogFormVisible = true
        this.$http({
              url: this.$http.adornUrl(`/sys/shop/selectTypeById/${data.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
               // console.log(data.data.name)
                this.form.name = data.data.name
                this.form.pid= data.data.pid
                this.form.id = data.data.id
              }
            })

    },
    callOf(form){
      this.dialogFormVisible = false
      if (this.$refs['refname']!==undefined) {
      this.$refs['refname'].resetFields();
      }
    },
    // renderContent自定义方法
    renderContent(createElement, { node, data, store }) {
      let _this = this;
        return createElement(
                'span',
                {
                    style:{
                        'flex':'1',
                        'display':'flex',
                        'align-items':'center',
                        'justify-content':'space-between',
                        'font-size':'14px',
                        'padding-right':'8px'
                    }
                },
                [
                    createElement(
                            'span',
                            {},
                            [
                                createElement('span',node.label)
                            ]
                    ),
                    createElement(
                            'span',
                            {},
                            [
                                createElement('el-button',
                                        {
                                            style:{
                                                'font-size':' 12px'
                                            },
                                            attrs:{
                                                'type':'text'
                                            },
                                            on:{
                                                click:function(){
                                                    _this.menuAdd(node,data,store)
                                                }
                                            },
                                            domProps: {
                                                innerHTML: '增加'
                                            }
                                        }
                                ),
                                createElement('el-button',
                                        {
                                            style:{
                                                'font-size':' 12px'
                                            },
                                            attrs:{
                                                'type':'text'
                                            },
                                            on:{
                                                click:function(){
                                                    _this.menuRemove(node,data,store);
                                                }
                                            },
                                            domProps: {
                                                innerHTML: '删除'
                                            }
                                        }
                                ),
                                createElement('el-button',
                                        {
                                            style:{
                                                'font-size':' 12px'
                                            },
                                            attrs:{
                                                'type':'text'
                                            },
                                            on:{
                                                click:function(){
                                                    _this.menuEdit(node,data,store);
                                                }
                                            },
                                            domProps: {
                                                innerHTML: '编辑'
                                            }
                                        }
                                )
                            ]
                    )
                ]
        );
    },
  
    }
};
</script>

