<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-lx-calendar"></i> 文章列表</el-breadcrumb-item>
                <el-breadcrumb-item>发布文章</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="plugins-tips">
                <div class="handle-box">
                    <el-input v-model="articleName" placeholder="请输入文章标题" class="handle-input mr10"></el-input>
                </div>
                <div>&nbsp;</div>
                <div class="handle-box">
                    <el-select v-model="articleCategory" placeholder="选择分类" class="handle-select mr10">
                        <el-option v-for="item in categoryArray" :key="item.categoryId" :label="item.categoryName" :value="item.categoryId"></el-option>
                    </el-select>
                </div>
            </div>
            <!--富文本编辑页面-->
            <quill-editor v-model="articleContent" ref="myTextEditor" :options="editorOption" @change="onChange" >
                <div id="toolbar" slot="toolbar">
                    <select class="ql-size">
                        <option value="small"></option>
                        <option selected></option>
                        <option value="large"></option>
                        <option value="huge"></option>
                    </select>
                    <!-- Add subscript and superscript buttons -->
                    <span class="ql-formats"><button class="ql-script" value="sub"></button></span>
                    <span class="ql-formats"><button class="ql-script" value="super"></button></span>
                    <span class="ql-formats"><button type="button" class="ql-bold"></button></span>
                    <span class="ql-formats"><button type="button" class="ql-italic"></button></span>
                    <span class="ql-formats"><button type="button" class="ql-blockquote"></button></span>
                    <span class="ql-formats"><button type="button" class="ql-list" value="ordered"></button></span>
                    <span class="ql-formats"><button type="button" class="ql-list" value="bullet"></button></span>
                    <span class="ql-formats"><button type="button" class="ql-link"></button></span>
                    <span class="ql-formats">
                                    <button type="button" @click="imgClick" style="outline:none">
                                    <svg viewBox="0 0 18 18"> <rect class="ql-stroke" height="10" width="12" x="3" y="4"></rect> <circle
                                            class="ql-fill" cx="6" cy="7" r="1"></circle> <polyline class="ql-even ql-fill"
                                                                                                    points="5 12 5 11 7 9 8 10 11 7 13 9 13 12 5 12"></polyline> </svg>
                                    </button>
                                  </span>
                    <span class="ql-formats"><button type="button" class="ql-video"></button></span>
                </div>
            </quill-editor>
            <el-button class="editor-btn" type="primary" @click="submit">提交</el-button>
        </div>
    </div>
</template>

<script>
    import 'quill/dist/quill.core.css';
    import 'quill/dist/quill.snow.css';
    import 'quill/dist/quill.bubble.css';
    import { quillEditor } from 'vue-quill-editor';
    import global_ from '../common/Global';
    export default {
        name: 'EditArticle',
        props:{
            value: {
                type: String
            },
            /*上传图片的地址*/
            uploadUrl: {
                type: String,
                default: ''
            },
            /*上传图片的file控件name*/
            fileName: {
                type: String,
                default: 'upload_file'
            },
            maxUploadSize:{
                type:Number,
                default: 1024 * 1024 * 500
            }
        },
        data(){
            return {
                articleId:'',
                articleName:'',
                articleCategory:'',
                articleContent: '',
                editorOption: {
                    placeholder: 'Hello World',
                    modules:{
                        toolbar: '#toolbar'
                    },
                },
                categoryArray:[]
            }
        },
        components: {
            quillEditor,
        },
        methods: {
            //获取参数
            getParam(){
              var articleId = this.$route.params.articleId;
              //获取文章详细信息
              var url = global_.serverUrl;
              var that = this;
              this.$axios.get(url+'/article/getArticleById?articleId='+articleId).then(res => {
                  that.articleId = articleId;
                  that.articleCategory = res.data.data.articleCategory;
                  that.articleContent = res.data.data.articleContent;
                  that.articleName = res.data.data.articleName
              });
            },

            //获取分类信息数据
            getCategoryInfo:function(){
                var url = global_.serverUrl;
                var that = this;
                this.$axios.get(url+"/category/getCategoryList").then(res => {
                    if (res.data.status==0){
                        that.categoryArray = res.data.data;
                    }
                })
            },
            onChange() {
                this.$emit('input', this.articleContent)
            },
            /*选择上传图片切换*/
            onFileChange(e){
                var fileInput = e.target;
                if(fileInput.files.length === 0){
                    return;
                }
                this.editor.focus();
                if (fileInput.files[0].size>this.maxUploadSize){
                    this.$alert('图片不能大于500KB\', \'图片尺寸过大',{
                        confirmButtonText:'确定',
                        type:'warning',
                    })
                }
                var data = new FormData(fileInput.files[0]);
                var url = global_.serverUrl;
                var save_url = url + '/article/articleUpload';
                data.append(this.fileName,fileInput.files[0]);
                this.$axios.post(save_url,data).then(res => {
                    if (res.data.status == 1){
                        this.$message.success(res.data.msg);
                    }else{
                        this.editor.insertEmbed(this.editor.getSelection().index, 'image', res.data);
                    }
                })
            },
            /*点击上传图片按钮*/
            imgClick(){
                //内存创建input File
                var form = document.createElement('form');
                form.enctype = 'multipart/form-data';
                form.method = 'post';
                var input = document.createElement('input');
                form.appendChild(input);
                input.type = 'file';
                input.name = 'upload_file';
                input.accept = 'image/jpeg,image/png,image/jpg,image/gif';
                input.onchange = this.onFileChange;
                input.click();
            },
            //提交表单数据
            submit(){
                var articleId = this.articleId;
                var articleName = this.articleName;
                var articleCategory = this.articleCategory;
                var articleContent = this.articleContent;
                if (articleName == ''){
                    this.$message.success('文章标题不能为空！');
                    return;
                }
                if (articleCategory == ''){
                    this.$message.success('文章分类不能为空！');
                    return;
                }
                if (articleContent == undefined){
                    this.$message.success('文章内容不能为空！');
                    return;
                }
                var userId = '';
                if(localStorage.getItem('userId')==''){
                    this.$message.success('用户未登录！请先登录');
                }else {
                    userId = localStorage.getItem('userId');
                }
                var article = {
                    articleId:articleId,
                    articleName:articleName,
                    articleCategory:articleCategory,
                    articleContent:articleContent,
                    publishUser:userId
                }
                var url = global_.serverUrl;
                var that = this;
                this.$axios.post(url+'/article/updateArticle',article).then(res => {
                    if (res.data.status==0){
                        this.$message.success('更新成功！');
                        setTimeout(function () {
                            that.$router.push({path:'/articleList'});
                        })
                    } else {
                        this.$message.success('更新失败！');
                    }
                })

            }
        },
        computed:{
            editor(){
                return this.$refs.myTextEditor.quill
            }
        },
        watch:{
            'value'(newVal,oldVal){
                if (this.editor){
                    if (newVal!==this.articleContent){
                        this.articleContent = newVal;
                    }
                }
            }
        },
        //Vue加载时执行的函数
        mounted() {
            this.getCategoryInfo();
            this.articleContent = this.value;
            this.getParam();
        }
    }
</script>
<style scoped>
    .editor-btn{
        margin-top: 20px;
    }
</style>