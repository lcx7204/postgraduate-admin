<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item><i class="el-icon-lx-calendar"></i> 推荐列表</el-breadcrumb-item>
                <el-breadcrumb-item>发布推荐</el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="plugins-tips">
                <div class="handle-box">
                    <el-input v-model="recommendTitle" placeholder="请输入推荐标题" class="handle-input mr10"></el-input>
                </div>
                <div>&nbsp;</div>
                <div class="handle-box">
                    <el-select v-model="recommendCategory" placeholder="选择分类" class="handle-select mr10">
                        <el-option v-for="item in categoryArray" :key="item.categoryId" :label="item.categoryName" :value="item.categoryId"></el-option>
                    </el-select>
                </div>
                <div>&nbsp;</div>
                <div class="handle-box">
                    <el-input v-model="recommendContent" placeholder="请输入推荐简介" class="handle-input mr10"></el-input>
                </div>
                <div>&nbsp;</div>
                <div class="handle-box">
                    <el-input v-model="recommendDetail" type="textarea" placeholder="请输入推荐详情" class="handle-input mr10"></el-input>
                </div>
                <div>&nbsp;</div>
                <el-form>
                    <el-form-item label="图标及图片">
                        <el-upload
                                class="upload-demo"
                                id="recommend_src"
                                ref="upload"
                                name="upload_file"
                                :action="uploadUrl"
                                :limit="1"
                                :on-exceed="onExceed"
                                :before-upload="beforeUpload"
                                :on-preview="handlePreview"
                                :on-remove="handleRemove"
                                :file-list="fileList"
                                @change="onFileChange">
                            <el-button slot="trigger" size="small" type="primary">上传图片</el-button>
                            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                        </el-upload>
                    </el-form-item>
                </el-form>

                <template>
                    <img :src="dialogImageUrl" width="200px">
                </template>

                <div>&nbsp;</div>
                <div class="handle-box">
                    <el-input v-model="companyName" placeholder="请输入出版社（主体）名称" class="handle-input mr10"></el-input>
                </div>
                <div>&nbsp;</div>
                <div class="handle-box">
                    <el-input v-model="author" placeholder="请输入作者（负责人）名称" class="handle-input mr10"></el-input>
                </div>
                <div>&nbsp;</div>
                <div class="handle-box">
                    <el-input v-model="price" placeholder="请输入书籍价格" class="handle-input mr10"></el-input>
                </div>
                <div>&nbsp;</div>
            </div>

            <el-button class="editor-btn" type="primary" @click="submit">提交</el-button>
        </div>
    </div>
</template>

<script>
    import global_ from '../common/Global';
    export default {
        name: "AddRecommend",
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
                recommendTitle: '',
                recommendCategory: '',
                recommendContent: '',
                recommendDetail:'',
                companyName:'',
                author:'',
                price:'',
                recommendIcon:'',
                categoryArray:[],
                //文件上传参数
                fileList: [],
                dialogImageUrl:'',
            }
        },
        methods:{
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
            //上传图片
            /*选择上传图片切换*/
            onFileChange(e){
                var fileInput = e;
                if(fileInput.length === 0){
                    return;
                }
                if (fileInput.size>this.maxUploadSize){
                    this.$alert('图片不能大于500KB\', \'图片尺寸过大',{
                        confirmButtonText:'确定',
                        type:'warning',
                    })
                }
                var data = new FormData(fileInput);
                var url = global_.serverUrl;
                var save_url = url + '/recommend/recommendUpload';
                data.append(this.fileName,fileInput);
                this.$axios.post(save_url,data).then(res => {
                    if (res.data.status == 1){
                        this.$message.success(res.data.msg);
                    }else{
                        if (res.data != ''){
                            this.dialogImageUrl = res.data.dialogImageUrl;
                            this.recommendIcon = res.data.recommendIcon;
                            this.$message({
                                type: 'info',
                                message: '图片上传成功',
                                duration: 6000
                            });
                            //将图片的名称保存下来存放到数据库
                        }else {
                            this.$message({
                                type: 'info',
                                message: '图片上传失败',
                                duration: 6000
                            });
                        }
                    }
                })
            },
            //上传图片成功的钩子函数
            handleSuccess(res, file) {
                console.log(res);
                this.$message({
                    type: 'info',
                    message: '图片上传成功',
                    duration: 6000
                });
                if (file.response.success) {
                    this.editor.picture = file.response.message; //将返回的文件储存路径赋值picture字段
                }
            },
            //删除文件之前的钩子函数
            handleRemove(file, fileList) {
                this.$message({
                    type: 'info',
                    message: '已删除原有图片',
                    duration: 6000
                });
            },
            //点击列表中已上传的文件事的钩子函数
            handlePreview(file) {
            },
            //上传的文件个数超出设定时触发的函数
            onExceed(files, fileList) {
                this.$message({
                    type: 'info',
                    message: '最多只能上传一个图片',
                    duration: 6000
                });
            },
            beforeUpload(file) {
                this.onFileChange(file);
                return false;
            },
            //提交保存
            submit(){
                var recommendTitle = this.recommendTitle;
                var recommendCategory = this.recommendCategory;
                var recommendContent = this.recommendContent;
                var recommendDetail = this.recommendDetail;
                var recommendIcon = this.recommendIcon;
                var companyName = this.companyName;
                var author = this.author;
                var price = this.price;

                var recommend = {
                    recommendTitle:recommendTitle,
                    recommendCategory:recommendCategory,
                    recommendContent:recommendContent,
                    recommendDetail:recommendDetail,
                    recommendIcon:recommendIcon,
                    companyName:companyName,
                    author:author,
                    price:price
                }

                var url = global_.serverUrl;
                var that = this;
                this.$axios.post(url+'/recommend/addRecommend',recommend).then(res => {
                    if (res.data.status==0){
                        this.$message.success('发布成功！');
                        setTimeout(function () {
                            that.$router.push({path:'/recommendList'});
                        })
                    } else {
                        this.$message.success('发布失败！');
                    }
                })
            }
        },
        //Vue加载时执行的函数
        mounted() {
            this.getCategoryInfo();
        }
    }
</script>

<style scoped>
    .editor-btn{
        margin-top: 20px;
    }
</style>