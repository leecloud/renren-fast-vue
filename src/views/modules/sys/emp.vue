<template>
    <div class="mod-emp">
        <!--顶部form表单-->
        <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
            <el-form-item>
                <el-input v-model="dataForm.name" placeholder="员工姓名" clearable></el-input>
            </el-form-item>
            <el-form-item>
                <el-button @click="getDataList()">查询</el-button>
                <el-button v-if="isAuth('sys:user:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
                <el-button v-if="isAuth('sys:user:delete')" type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
            </el-form-item>
        </el-form>
        <!--table表单-->
        <el-table 
            :data="dataList"
            border
            v-loading="dataListLoading"
            @selection-change="selectionChangeHandle"
            style="width: 100%;">
            <!--选择框-->
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
                label="姓名">
            </el-table-column>
            <el-table-column
                prop="depName"
                header-align="center"
                align="center"
                label="部门">
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
                label="创建时间">
            </el-table-column>
            <el-table-column
                prop="updateTime"
                header-align="center"
                align="center"
                label="修改时间">
            </el-table-column>
            <el-table-column
                prop="delFlag"
                header-align="center"
                align="center"
                label="删除标识">
                <template slot-scope="scope">
                <el-tag v-if="scope.row.delFlag === true" size="small" type="danger">是</el-tag>
                <el-tag v-else size="small">否</el-tag>
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
        <!--分页-->
        <el-pagination
            @size-change="sizeChangeHandle"
            @current-change="currentChangeHandle"
            :current-page="pageIndex"
            :page-sizes="[10, 20, 50, 100]"
            :page-size="pageSize"
            :total="totalPage"
            layout="total, sizes, prev, pager, next, jumper">
        </el-pagination>
        <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
    
    </div>
</template>
<script>
import AddOrUpdate from './emp-add-or-update'
export default {
    data(){
        return {
            dataForm:{
                name: ''
            },
            dataList: [],
            dataListLoading: false,
            pageIndex: 1,
            pageSize: 10,
            totalPage: 0,
            dataListSelections: [],
            addOrUpdateVisible: false
        }
    },
     // 引入的组件
    components: {
      AddOrUpdate
    },
    // 一进页面就会加载的方法
    activated(){
        this.getDataList()
    },
    // 方法
    methods: {
        getDataList(){
            this.dataListLoading = true
            this.$http({
                url: this.$http.adornUrl('/sys/emp/list'),
                method: 'get',
                params: this.$http.adornParams({
                    'page': this.pageIndex,
                    'limit': this.pageSize,
                    'name': this.dataForm.name
                })
            }).then(({data}) => {
                if (data && data.code === 0) {
                    console.log(data)
                    this.dataList = data.data.list
                    this.totalPage = data.data.totalPage
                } else {
                    this.dataList = []
                    this.totalPage = 0
                }
            this.dataListLoading = false
            })
        },
        // 每页数
        sizeChangeHandle(val){
            this.pageSize = val
            this.pageIndex = 1
            // 获取数据刷新
            this.getDataList()
        },
        // 当前页
        currentChangeHandle(val){
            this.pageIndex = val
            this.getDataList()
        },
        // 多选
        selectionChangeHandle(val){
            this.dataListSelections = val
        },
         // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
        // 删除
        deleteHandle(id){
            var empIds = id ? [id] : this.dataListSelections.map(item=>{
                return item.id
            })
            this.$confirm(`确定对[id=${empIds.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
            }).then(()=>{
                this.$http({
                    url: this.$http.adornUrl('/sys/emp/delete'),
                    method: 'post',
                    data: this.$http.adornData(empIds, false)
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
        }
    }
    
}
</script>

