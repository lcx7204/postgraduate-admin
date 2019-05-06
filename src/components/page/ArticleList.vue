<template>
    <div class="table">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-lx-cascades"></i> 文章列表</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
                <el-button type="primary" icon="delete" class="handle-del mr10">批量删除</el-button>
                <el-button type="primary" icon="add" class="handle-del mr10" @click="addArticle">添加</el-button>
                <el-input v-model="select_word" placeholder="筛选关键词" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="search" @click="search">搜索</el-button>
            </div>
            <el-table :data="articleData" style="width: 100%">
                <el-table-column type="selection" width="55" align="center">
                </el-table-column>
                <el-table-column prop="articleName" label="文章标题" width="200">
                </el-table-column>
                <el-table-column prop="articleCategory" label="文章分类" width="200">
                </el-table-column>
                <el-table-column prop="publishUser" label="作者" sortable width="150">
                </el-table-column>
                <el-table-column prop="publishDate" label="发布日期" sortable width="150" :formatter="dateFormat">
                </el-table-column>
                <el-table-column prop="praiseNum" label="点赞数" sortable width="100">
                </el-table-column>
                <el-table-column label="操作" width="180" align="center">
                    <template slot-scope="scope">
                        <el-button type="text" icon="el-icon-edit" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                        <el-button type="text" icon="el-icon-delete" class="red" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
                <el-pagination background @current-change="handleCurrentChange" :current-page="pageNum" :page-size="pageSize" layout="prev, pager, next" :total="totalCount">
                </el-pagination>
            </div>
        </div>


    </div>
</template>

<script>
    import global_ from '../common/Global';
    import {formatDate} from "../common/date";

    export default {
        name: 'ArticleList',
        data() {
            return {
                articleData:[],     //存放数据
                select_word: '',    //筛选关键字
                form: {
                    articleName: '',
                    articleCategory: '',
                    publishUser: '',
                    publishDate:'',
                    praiseNum:'',
                    articleId:''
                },
                //分页信息
                pageNum:1,              //当前页
                totalCount:0,           //记录总数
                pageSize:10,             //每页显示条数
            }
        },
        computed: {
        },
        methods: {
            //获取文章列表（首页的）
            getArticleList:function(){
                var that = this;
                var url = global_.serverUrl;
                var pageNum = this.pageNum;
                var pageSize = this.pageSize;
                this.$axios.get(url+"/article/getArticleList?pageNum="+pageNum+"&pageSize="+pageSize).then(res => {
                    that.articleData = res.data.data.list;
                    that.totalCount = res.data.data.total;
                })
            },
            //时间格式化
            dateFormat:function(row){
              var date = new Date(row.publishDate);
              return formatDate(date,'yyyy-MM-dd hh:mm')
            },
            //添加按钮
            addArticle:function(){
                this.$router.push({path:'/addArticle'});
            },
            //搜索
            search() {

            },
            //分页跳转
            handleCurrentChange:function (val) {
                this.pageNum = val;
                this.getArticleList();
            },
            //编辑页面
            handleEdit:function (index,row) {
                this.$router.push({path:'/editArticle/'+row.articleId});
            },
            //删除
            handleDelete:function (index,row) {
                var articleId = row.articleId;
                var url = global_.serverUrl;
                var that = this;
                this.$axios.delete(url+'/article/deleteArticle?articleId='+articleId).then(res => {
                    if (res.data.status==0){
                        this.$message.success(res.data.msg);
                        that.getArticleList();
                    }
                    if (res.data.status==1){
                        this.$message.success(res.data.msg);
                    }
                })
            }
        },
        mounted() {
            this.getArticleList();
        }
    }

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
    .del-dialog-cnt{
        font-size: 16px;
        text-align: center
    }
    .table{
        width: 100%;
        font-size: 14px;
    }
    .red{
        color: #ff0000;
    }
    .mr10{
        margin-right: 10px;
    }
</style>