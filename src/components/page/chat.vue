<template>
    <div>
        <div class="leftbar">
            <ul>
                <li><i class="fas fa-user"></i></li>
                <li><i class="fas fa-users"></i></li>
                <li><i class="fas fa-smile"></i></li>
                <li><i class="fas fa-envelope"></i></li>
                <li><i class="fas fa-bell"></i></li>
                <li><i class="fas fa-calendar-alt"></i></li>
                <li><i class="fas fa-power-off"></i></li>
            </ul>
        </div>
        <div class="container">
            <div class="chatbox">
                <div class="chatleft">
                    <div class="center">
                        <ul>
                            <li v-for="(item,index) in userList" v-bind:class="{ active: index==selectItem }" :id="item.userId" class="active" @click="getUserChat(item.userId)">
                                <img style="width:50px;height:50px;border-radius: 50%; vertical-align: middle;" :src="item.avatorUrl">
                                <span style="margin-left: 10px;">{{item.nickName}}</span>
                            </li>
                        </ul>
                    </div>
                </div>
                <div class="chatright">
                    <div class="center">
                        <ul>
                            <li v-bind:class="{msgright:item.sendId==sendId}" class="msgleft" v-for="(item,index) in chatList">
                                <img style="border-radius: 20px; vertical-align: top;" src="http://placehold.it/40x40">
                                <p class="msgcard">{{item.chatContent}}</p>
                            </li>
                        </ul>
                    </div>
                    <div class="footer">
                        <textarea maxlength="800" rows="5" cols="40" style="width: 100%; resize: none; border: none; " placeholder="请在此输入要发送的内容..." v-model="chatContent"></textarea>
                        <button class="sendbtn" @click="sendMsg">发送</button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    import global_ from '../common/Global';
    import {formatDate} from "../common/date";

    export default {
        name: "chat",
        data(){
            return{
                chatContent:'',
                selectItem:0,
                userList:[],
                chatList:[],
                accessId: '',
                send:''
            }
        },
        methods:{
            //获取注册用户列表
            getUserList:function () {
                var that = this;
                var url = global_.serverUrl;
                this.$axios.get(url+"/chat/getUserList").then(res => {
                    that.userList = res.data.data
                })
            },
            //获取聊天记录
            getUserChat:function (val) {
                var that = this;
                var url = global_.serverUrl;
                this.accessId = val;
                this.$axios.get(url+"/chat/getUserChat?userId="+val).then(res => {
                    that.chatList = res.data.data;
                })
            },
            //发送消息
            sendMsg:function () {
                var chatContent = this.chatContent;
                if (chatContent == ''){
                    this.$message.success('请输入内容后提交！');
                    return;
                }
                var userId = localStorage.getItem("userId");
                var chat = {
                    sendId:userId,
                    accessId:this.accessId,
                    chatContent:this.chatContent
                }
                var that = this;
                var url = global_.serverUrl;
                this.$axios.post(url+"/chat/addChat",chat).then(res=>{
                    that.getUserChat(that.accessId);
                    that.chatContent = "";
                })
            },
            get(){
                this.getUserChat(this.accessId);
            }
        },
        mounted() {
            this.getUserList();
            this.sendId = localStorage.getItem("userId");
            this.timer = setInterval(this.get, 3000);
        }
    }
</script>

<style scoped>
    .active{
        background-color: #00d1b2;
    }
    li {
        list-style: none;
    }
    input {
        border-radius: 5px;
    }
    .leftbar ul {
        padding: 0px;
    }
    .leftbar li {
        padding: 10px;
    }
    .leftbar {
        background-color: #212e3e;
        width: 50px;
        height: 100%;
        color:grey;
        padding: 0px;
        text-align: center;
        position: absolute;
    }
    .container {
        height: 100%;
        width: 100%;
        background:  linear-gradient(#1f94bf 20%,#edf5f8 0, #edf5f8 80%, #1f94bf 0);
        display: flex;
        margin: 50px 0;
    }
    .chatbox {
        margin: auto;
        height: 70%;
        width: 70%;
        transform: translateY(-10%);
        box-shadow: 0 0 0 1px gray;
        display: flex;
    }
    .chatleft {
        background-color: #ffffff;
        width: 25%;
        left: 0;
        height: 100%;
    }
    .chatleft .top {
        height: 10%;
        color: grey;
        background-color: #F7F9F9;
        display: flex;
        align-items: center;
        padding-left: 20px;
    }
    .searchbtn {
        height: 36px;
        width: 36px;
        border-radius: 18px;
        background-color: #1f94bf;
        color: #ECF0F1;
        margin-left: 10px;
    }
    .searchbtn:hover {
        cursor: pointer;
    }
    .chatleft .center {
        overflow-y: scroll;
        height: 90%;
    }
    .chatleft .center ul{
        padding-left: 10px;
    }
    .chatleft .center li{
        margin: 10px;
    }
    .chatright {
        background-color: #ffffff;
        width: 75%;
        height: 100%;
    }
    .chatright .top {
        height: 10%;
        display: flex;
        align-items: center;
        padding-left: 30px;
    }
    .chatright .center {
        background-color: #edf5f8;
        height: 65%;
        overflow-y: scroll;
    }
    .chatright .center ul {
        padding: 10px;
    }
    .chatright .center li {
        margin: 10px;
        width: 100%;
    }
    .chatright .center p{
        display: inline-block;
    }
    .msgcard {
        margin: 0 10px 0 10px;
        background-color: white;
        border-radius: 10px;
        padding: 10px;
        max-width: 60%;
    }
    .msgleft {
        float: left;
    }
    .msgright {
        width: 100%;
        text-align: right;
    }
    .msgright img{
        float: right;
    }
    .chatright .footer {
        height: 25%;
        background-color: #FBFCFC;
        text-align: right;
    }
    .sendbtn {
        height: 40px;
        width: 80px;
        border-radius: 10px;
        background-color: #58D68D;
        color: white;
        font-weight: bold;
        margin:10px 20px 0 0;
    }
</style>