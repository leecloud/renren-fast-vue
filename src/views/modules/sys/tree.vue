<template>
    <el-tree
        :props="props"
        node-key="id"
        ref="tree"
        highlight-current
        :load="loadNode"
        lazy
        show-checkbox
        @node-click="handleNodeClick"
        :render-content="renderContent">
    </el-tree>
  
</template>
<script>
export default {
    data(){
        return {
            props: {
                label: 'name',
                children: [],
                isLeaf: 'leaf' //leaf的作用是是否显示下拉箭头
            }
        }
    },
    methods: {
        // 加载根节点
        requestTree(resolve){
            this.$http({
               url: this.$http.adornUrl('/sys/tree/firstTree'),
               method: 'get'
           }).then(({data})=>{
               if(data){
                   this.leaf = false
                   resolve(data.data)
               }
           })
           
           
        },
        // // 异步树点击事件
        handleNodeClick(data){
            console.log(data)
        },
        // // 异步树子节点加载逻辑
        loadNode(node,resolve){
            // 一级节点
            if(node.level === 0){
                this.requestTree(resolve)
            }
            // 其他节点处理
            if(node.level >= 1){
                this.getIndex(node,resolve)
            }
        },
         // 根据上级节点异步加载节点方法
        getIndex(node,resolve){
            this.$http({
                url: this.$http.adornUrl(`/sys/tree/getByPid/${node.data.id}`),
                method: 'get',
                params: this.$http.adornParams()
           }).then(({data})=>{
               if(data){
                   resolve(data.data)
               }
           })
        },
        menuAdd(node,data,store){
            alert(data.id)
        },
        // 自定义节点的操作
        renderContent(createElement, { node, data, store }){
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
        }
    }
}
</script>

