<template>
    <div class="table">
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-lx-cascades"></i> 基础表格</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
                <el-button type="primary" icon="delete" class="handle-del mr10" @click="delAll">批量删除</el-button>
                <el-button type="primary" icon="add" class="handle-del mr10" @click="addArticle">添加</el-button>
                <el-input v-model="select_word" placeholder="筛选关键词" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="search" @click="search">搜索</el-button>
            </div>
            <el-table :data="data" border class="table" ref="multipleTable" @selection-change="handleSelectionChange">
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <el-table-column prop="articleName" label="文章标题" width="120">
                </el-table-column>
                <el-table-column prop="articleCategory" label="文章分类" width="120">
                </el-table-column>
                <el-table-column prop="publishUser" label="作者" sortable width="150">
                </el-table-column>
                <el-table-column prop="publishDate" label="发布日期" sortable width="150":formatter="formatter">
                </el-table-column>
                <el-table-column prop="praiseNum" label="点赞数" sortable width="50">
                </el-table-column>
                <el-table-column label="操作" width="180" align="center">
                    <template slot-scope="scope">
                        <el-button type="text" icon="el-icon-edit" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                        <el-button type="text" icon="el-icon-delete" class="red" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
                    </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
                <el-pagination background @current-change="handleCurrentChange" layout="prev, pager, next" :total="1000">
                </el-pagination>
            </div>
        </div>


    </div>
</template>

<script>
    import global_ from '../common/Global';

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
            }
        },
        computed: {

        },
        methods: {
            //获取文章列表（首页的）
            getArticleList:function(){
                var url = global_.serverUrl;
                var that = this;
                var pageNum = 1;
                var pageSize = 5;

                this.$axios.get(url+"/article/getArticleList?pageNum=1&pageSize=2").then(res => {
                    console.log(res.data.data);
                })
            },
            search() {

            },
            formatter(row, column) {
                return row.address;
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