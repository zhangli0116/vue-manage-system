<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-lx-cascades"></i> 厂家信息
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
                <el-button
                    type="primary"
                    icon="el-icon-delete"
                    class="handle-del mr10"
                    @click="delAllSelection"
                    disabled
                >批量删除</el-button>
                <el-select v-model="query.address" placeholder="地址" class="handle-select mr10" disabled>
                    <el-option key="1" label="广东省" value="广东省"></el-option>
                    <el-option key="2" label="湖南省" value="湖南省"></el-option>
                </el-select>
                <el-input v-model="query.name" placeholder="厂家名" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button>
                <el-button type="primary" icon="el-icon-circle-plus-outline" @click="handleInsert">添加</el-button>
                <el-button type="primary" icon="el-icon-folder-add" disabled>文件导入</el-button>
            </div>
            <el-table
                :data="tableData"
                border
                class="table"
                ref="multipleTable"
                header-cell-class-name="table-header"
                @selection-change="handleSelectionChange"
            >
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <el-table-column label="序号" width="55">
                    <template slot-scope="scope">{{ scope.$index +1 }}</template>
                </el-table-column>
                <!-- <el-table-column prop="id" label="ID" width="55" align="center"></el-table-column> -->
                </el-table-column>
                <el-table-column prop="name" label="厂家名"></el-table-column>
                <!-- <el-table-column label="账户余额">
                    <template slot-scope="scope">￥{{scope.row.money}}</template>
                </el-table-column> -->
                <!-- <el-table-column label="头像(查看大图)" align="center">
                    <template slot-scope="scope">
                        <el-image
                            class="table-td-thumb"
                            :src="scope.row.thumb"
                            :preview-src-list="[scope.row.thumb]"
                        ></el-image>
                    </template>
                </el-table-column> -->
                <el-table-column prop="place" label="地址"></el-table-column>
                <el-table-column prop="personname" label="客户经理"></el-table-column>
                <el-table-column prop="phone1" label="电话"></el-table-column>
                <el-table-column prop="phone2" label="其他联系方式"></el-table-column>
                <el-table-column prop="remark" label="备注"></el-table-column>
                <!-- <el-table-column label="状态" align="center">
                    <template slot-scope="scope">
                        <el-tag
                            :type="scope.row.state==='成功'?'success':(scope.row.state==='失败'?'danger':'')"
                        >{{scope.row.state}}</el-tag>
                    </template>
                </el-table-column> -->

                <!-- <el-table-column prop="date" label="注册时间"></el-table-column> -->
                <el-table-column label="操作" width="180" align="center">
                    <template slot-scope="scope">
                        <el-button
                            type="text"
                            icon="el-icon-edit"
                            @click="handleEdit(scope.$index, scope.row)"
                        >编辑</el-button>
                        <el-button
                            type="text"
                            icon="el-icon-delete"
                            class="red"
                            @click="handleDelete(scope.$index, scope.row)"
                        >删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页模块 -->
            <div class="pagination">
                <el-pagination
                    background
                    layout="total, prev, pager, next"
                    :current-page="query.pageIndex"
                    :page-size="query.pageSize"
                    :total="pageTotal"
                    @current-change="handlePageChange"
                ></el-pagination>
            </div>
        </div>

        <!-- 编辑 -->
        <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="100px">
                <el-form-item label="用户名">
                    <el-input v-model="form.name"></el-input>
                </el-form-item>
                <el-form-item label="地址">
                    <el-input v-model="form.place" name="place"></el-input>
                </el-form-item>
                <el-form-item label="客户经理">
                    <el-input v-model="form.personname"></el-input>
                </el-form-item>
                <el-form-item label="电话">
                    <el-input v-model="form.phone1"></el-input>
                </el-form-item>
                <el-form-item label="其他联系方式">
                    <el-input v-model="form.phone2"></el-input>
                </el-form-item>
                <el-form-item label="备注">
                    <el-input v-model="form.remark"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
        </el-dialog>


    </div>
</template>

<script>
//import { fetchData } from '../../api/index';
//封装的一个axios对象
import request from '@/utils/request';

export default {
    name: 'basetable',
    data() {
        return {
            //请求参数
            query: {
                name: '',
                //当前页数
                pageIndex: 1,
                //每页数据量    
                pageSize: 5
            },
            //总数据量，分页组件会自动分配,初始化0
            pageTotal: 0,
            //总页数
            pageCount:0,
            //表格数据（每一页的）
            tableData: [],
            multipleSelection: [],
            delList: [],
            editVisible: false,
            //表单数据
            form: {},
            idx: -1,
            id: -1
        };
    },
    //使用哪个阶段生命周期更合适？ created ? mounted?
    mounted() {
        //this.getData();
        this.initData();
    },
    methods: {
        //表格数据初始化
        initData(){
            var _url = this.HOST + '/fangxing/getFactorysAll'
            request({
                //默认application/json类型
                // headers: {
                //   'Content-Type': 'application/json'
                // },
                // // //向服务器发送数据前,转换json格式数据
                // transformRequest: [data => {
                //   var data = JSON.stringify(data)
                //   return data
                // }],
                url:_url,
                method:'post',
                //请求数据
                data:{
                    name:this.query.name,
                    currentPage:this.query.pageIndex,
                    pageSize:this.query.pageSize
                }
            }).then(res =>{
                //获得后台分页数据
                this.tableData = res.data
                //数据量
                this.pageTotal = res.pageTotal
                //总页数
                this.pageCount = Math.ceil(this.pageTotal/this.query.pageSize)
            }).catch(res =>{
                console.log(res)
            })

        },
        // 获取 easy-mock 的模拟数据
        // getData() {
        //     fetchData(this.query).then(res => {
        //         console.log(res);
        //         this.tableData = res.list;
        //         this.pageTotal = res.pageTotal || 50;
        //     });
        // },
        // 触发搜索按钮
        handleSearch() {
            // this.$set(this.query, 'pageIndex', 1);
            // this.getData();
            this.$set(this.query,'pageIndex',1);
            this.initData();
        },

        // 删除操作
        handleDelete(index, row) {
            // 二次确认删除
            this.$confirm('确定要删除吗？', '提示', {
                type: 'warning',
                confirmButtonText: '确定',
                cancelButtonText: '取消',
            }).then(() => {    
                    var _url = this.HOST + '/fangxing/deleteFactory';
                    request({
                        url:_url,
                        method:'post',
                        data:row
                    }).then(res =>{
                        if(res.deleteStatus == 1){
                            this.$message.success('删除成功');
                            //js数组语法 splice
                            //this.tableData.splice(index, 1); 
                            //保证分页模块同步
                            this.pageTotal = this.pageTotal - 1; 
                            console.log(this.pageCount);
                            console.log(this.pageTotal);
                            //BUG 最后一页数据清空后，需要回退到上一页
                            //如果当前页是最后一页
                            if(this.query.pageIndex == this.pageCount && this.pageTotal % this.query.pageSize ==0 ){
                                //当前页-1
                                this.$set(this.query,'pageIndex',this.query.pageIndex-1);
                            }
                            //刷新 
                            this.initData();
                        }
                    }).catch(res =>{
                        console.log(res);
                    })


                }).catch(() => {
                    this.$message({
                        type:'info',
                        message:'已取消删除'
                    })
                });
        },
        // 多选操作
        handleSelectionChange(val) {
            this.multipleSelection = val;
        },
        //批量删除
        delAllSelection() {
            const length = this.multipleSelection.length;
            let str = '';
            this.delList = this.delList.concat(this.multipleSelection);
            for (let i = 0; i < length; i++) {
                str += this.multipleSelection[i].name + ' ';
            }
            this.$message.error(`删除了${str}`);
            this.multipleSelection = [];
        },
        // 编辑操作
        handleEdit(index, row) {
            this.idx = index;
            this.form = row;
            this.editVisible = true;
        },
        // 保存编辑或添加
        saveEdit() {
            var _this = this;
            var _idx = this.idx;
            //弹窗消失
            this.editVisible = false;
            //动态更新前端数据，不经过后台,这句好像不用加，前端数据已经动态变化了
            //this.$set(this.tableData, this.idx, this.form); 
            //如果有行数表示是修改数据，否则为添加数据
            if(this.idx >= 0){
                var _url = this.HOST + '/fangxing/updateFactory'
                request({
                    url:_url,
                    method:'post',
                    data:this.form,
                    //这里我们进行 表单垃圾处理
                    transformResponse: [ data => {
                    //初始化
                    _this.idx = -1;
                    _this.form = {};
                    //
                    return JSON.parse(data);
                   }]
                }).then(res =>{
                    console.log(res.updateStatus);
                    //消息提示
                    res.updateStatus==1 && this.$message.success(`修改第 ${_idx + 1} 行成功`);
                }).catch(res =>{
                    console.log(res)
                    this.$message.info('服务繁忙,修改失败');
                    
                }) 
            }else{
                var _url = this.HOST +'/fangxing/insertFactory'
                request({
                    url:_url,
                    method:'post',
                    data:this.form
                }).then(res =>{
                    res.insertStatus==1 && this.$message.success('添加成功');
                    this.initData();
                }).catch(res=>{
                    this.$message.info('服务繁忙，添加失败');
                })
            }
           

        },
        //添加
        handleInsert(){
            this.editVisible = true;
        },
        // 当前页改变时触发current-change事件,val为当前页数
        handlePageChange(val) {
            //更高级的用法,会同步渲染画面
            this.$set(this.query, 'pageIndex', val);
            this.initData();
        }
    }
};
</script>

<style scoped>
.handle-box {
    margin-bottom: 20px;
}

.handle-select {
    width: 120px;
}

.handle-input {
    width: 300px;
    display: inline-block;
}
.table {
    width: 100%;
    font-size: 14px;
}
.red {
    color: #ff0000;
}
.mr10 {
    margin-right: 10px;
}
.table-td-thumb {
    display: block;
    margin: auto;
    width: 40px;
    height: 40px;
}
</style>
