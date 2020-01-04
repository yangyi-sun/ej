<template>
    <div>
        <!-- 按钮-->
        <el-button type="primary" size="small" @click="toAddHandler">添加</el-button>
        <el-button type="danger" size="small">批量删除</el-button>
        <!--/按钮结束-->
        <!-- 表格-->
        <el-table :data="order">
            <el-table-column prop="id" label="订单编号"></el-table-column>
            <el-table-column prop="name" label="下单时间"></el-table-column>
            <el-table-column prop="total" label="总价"></el-table-column>
            <el-table-column prop="status" label="状态"></el-table-column>
            <el-table-column prop="waiterId" label="顾客id"></el-table-column>
            <el-table-column label="操作">
                <template v-slot="slot">
                    <a href="" @click.prevent="toDeleteHandler(slot.row.id)">删除</a>
                    <a href="" @click.prevent="toUpdateHandler(slot.row)">修改</a>
                </template>
            </el-table-column>
        </el-table>
        <!-- /表格结束-->
        <!-- 分页-->
        <el-pagination layout="prev, pager, next" :total="50"></el-pagination>
        <!-- /分页结束-->
        <!-- 模态框-->
        <el-dialog 
            :title="title"
            :visible.sync="visible"
            width="30%">
            ---{{form}}
            <el-form :model="form" label-width="80px">
                <el-form-item label="订单编号">
                    <el-input v-model="form.id"></el-input>
                </el-form-item>
                <el-form-item label="总价">
                    <el-input v-model="form.total"></el-input>
                </el-form-item>
                <el-form-item label="状态">
                    <el-input type="textarea" v-modal="textarea" placeholder="请输入" :rows="2"></el-input>
                </el-form-item>
                <el-form-item label="顾客id" >
                    <el-input v-model="form.waiterId"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button size="small" @click="closeModalHandler">取 消</el-button>
                <el-button size="small" type="primary" @click="submitHandler">确 定</el-button>
            </span>
            
        </el-dialog>
        <!-- /模态框结束-->
    </div>
</template>

<script>
import request from '@/utils/request'
import querystring from 'querystring'
export default {
    //用于存放网页中需要调用的方法
    methods:{
        //数据重载
        loadData(){
            //ue实例创建完毕
            let url="http://localhost:6677/order/findAll"
            request.get(url).then((response)=>{
            //this指向当前vue实例（message中的数据、data中的数据）
            //箭头函数将查询结果设置到customers中this指向外部函数的this
            this.order=response.data;
        })
        },
        submitHandler(){
            //this.from 对象 --- 字符串 ---> 后台 {type:'customer',age:2}
            //json字符串 '{"type":"customer","age":12}'
            //request.post(url,this.form)
            //查询字符串: type=customer&age=12
            //通过request与后台进行交互，并且携带参数
            let url = "http://localhost:6677/order/save"
            request({
                url,
                method:"POST",
                headers:{
                    "Content-Type":"application/x-www-form-urlencoded"
                },
                data:querystring.stringify(this.form)
            }).then((response)=>{
                //请求结束
                this.closeModalHandler();
                //刷新
                this.loadData();
                //提示
                this.$message({
                    type:"success",
                    message:response.message
                })
            })
        },
        toDeleteHandler(id){
            this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).then(() => {
                let url="http://localhost:6677/order/deleteById?id="+id;
                request.get(url).then((response)=>{
                    this.loadData();
                     this.$message({
                    type: 'success',
                    message: response.message
                });
                });
            })
        },
        toUpdateHandler(row){
           this.form=row;
           this.title="修改订单信息";
           this.visible=true;
        },
        closeModalHandler(){
            this.visible=false;
        },
        toAddHandler(){
             this.form={
                type:"order"
            }
            this.title = "录入订单信息";
            this.visible = true;
        }
    },
    //用于存放要向网页中显示的数据
    data(){
        return{
            title:"录入订单信息",
            visible:false,
            order:[],
            form:{
                type:"order"
            }
        }
    },
    created(){
        //页面加载出来
        //this为当前vue实例
        //将查询结果设置到customers中this指向外部函数的this
        this.loadData();
    }
}
</script>

<style scoped>

</style>
