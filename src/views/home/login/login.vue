<style lang="scss" scoped>
.bg{
    background-image:url('./bg2.jpg');
    width:100%;
    height:100%;
    background-size:100% 100%;
}
.language{
    position:absolute;
    top:20px;
    left:20%;
    height:20px;
    color:#fff;
    font-size:16px;
    a{
        color:#fff;
        padding:0 15px;
    }
}
.logo{
    width:257px;
    padding-top:50px;
    img{
        width:100%;
        height:100%;
        display:block;
    }
}
.login{
    width:350px;
    height:290px;
    margin-top:30px;
    background:#fff;
    border-radius:5px;
    .login-inner{
        margin:0 auto;
        width:280px;
        padding-top:20px;
        p{
            font-size:18px;
            color:#333;
            text-align:center;
        }
        a.now{
            margin:10px auto 0;
            width:100%;
            display:block;
            color:#199ed8;
            text-align:center;
            text-decoration:underline;
        }
    }
}
.demo-spin-icon-load{
        animation: ani-demo-spin 1s linear infinite;
    }
</style>
<template>
<div class="bg">
    <!--<div class="language">
        <a href="">中文</a>|
        <a href="">English</a>
    </div>-->
    <div class="logo container">
         <img src="./logo2.png" />
    </div>
    <div class="login container">
        <div class="login-inner">
            <p>用户登录</p>
            <Form ref="formLogin" :model="formLogin" :rules="ruleLogin">
                <FormItem prop="email">
                    <Input type="text" id="useremail" v-model="formLogin.email" placeholder="请输入邮箱">
                        <Icon type="android-mail" slot="prepend"></Icon>
                    </Input>
                </FormItem>
                <FormItem prop="password">
                    <Input type="password" id="userpassword" v-model="formLogin.password" placeholder="请输入密码">
                        <Icon type="ios-locked" slot="prepend"></Icon>
                    </Input>
                </FormItem>
                <div style="margin-top:18px;">
                    <div>
                        <template>
                            <Checkbox v-model="rememberPassword" @on-change="rememberPw">记住密码</Checkbox>
                            <a href="" style="float:right;color:#199ed8;">忘记密码？</a>
                        </template>
                    </div>
                    
                    <!-- <input type="checkbox" />
                    <span style="color:#666">记住密码</span> -->
                    
                </div>
                <FormItem style="width:280px;margin-top:10px;">
                    <Button type="primary" @click="login('formLogin')"  style="width:280px;" v-loading="load">登录</Button>
                </FormItem>
            </Form>
            <!--<router-link to="/register" class="now">立即注册</router-link>-->
            <a @click="clickBtn" href="javascript:;" class="now">三方登录</a>
        </div>
    </div>
</div>   
</template>
<script>
//引入登录方法
import {
    login
} from 'api/index.js'
import {setCookie,getCookie,delCookie} from '@/common/js/cookie.js'
export default{
    name:"login",
    data(){
            // 验证邮箱是否为空
            let validateLoginEmail = (rule, value, callback) => {
                if (!(/^(\w-*\.*)+@(\w-?)+(\.\w{2,})+$/.test(value))) {
                    return callback(new Error('邮箱不能为空或邮箱错误'));
                } else {
                    callback();
                }
            };
            // 验证密码是否为空
            let validateLoginPassword = (rule, value, callback) => {
                if (value === '') {
                    return callback(new Error('密码不能为空'));
                } else {
                    callback();
                }
            };
        return{
            rememberPassword: false,
            flag:true,
            load:false,
            formLogin: {
                    email: '',
                    password: ''
            },
            ruleLogin: {
                email: [{
                    validator: validateLoginEmail,
                    trigger: 'blur'
                }],
                password: [{
                    validator: validateLoginPassword,
                    trigger: 'blur'
                }]
            }
        }
    },
    // 实例创建时
    created() {
        this.watchCheckbox();
    },
    methods:{
        // 第三方登录
        clickBtn(){
            delCookie("userid");
            delCookie("email");
            let flag=true;
                if(flag){
                    flag=false;
                    login.jump().then((data)=> {
                    // console.log(data)
                    let url = data.data;
                    if(url) {
                        window.location.href = url;
                    }else{
                        this.$Message.error("错误")
                    }
                    flag=true;
                })
            }
            
        },
        watchCheckbox() {
            if(getCookie('email')) {
                console.log(typeof(getCookie('rememberPassword')));
                if(getCookie('rememberPassword')=='true') {
                    this.rememberPassword = Boolean(getCookie('rememberPassword'));
                    this.rememberPw(this.rememberPassword);
                    if(this.rememberPassword == true) {
                        this.formLogin = {
                            email: getCookie('email'),
                            password: getCookie('passWord')
                        }
                    }
                }
            }
        },
        // 记住密码
        rememberPw(val) {
            this.rememberPassword = val ? val : false;
            // console.log(this.rememberPassword);
        },
        login(name) {
            this.load=true;
            this.$refs[name].validate((valid) => {
                    if (valid) {
                // 调取登录方法，参数为：email，password
            let obj={
                 'passWord': this.formLogin.password,
                 'email':this.formLogin.email
            }
            if(this.formLogin.password!=''&&this.formLogin.email!=''){
                login.login(obj).then((data) => {
                        if(data.data==null || data.data=="null"){
                            this.$Message.error(data.msg)
                            this.load=false;
                        }else{
                            setCookie('email',data.data.email,null,null,"365");
                            setCookie('userid',data.data.userid,null,null,"365");
                            setCookie('passWord',obj.passWord,null,null,"365");
                            if(this.rememberPassword == true) {
                                setCookie('rememberPassword',this.rememberPassword,null,null,"365");
                            }else {
                               setCookie('rememberPassword',this.rememberPassword,null,null,"365"); 
                            }
                            this.$router.push('/admin');
                            console.log(data)
                            console.log(getCookie('userid'))
                            this.load=false;
                        }
                    }).catch((error)=>{
                        console.log(error)
                        this.load=false;
                    })
            }else{
                this.$Message.error("邮箱或密码为空！")
                this.load=false;
            }       
            }else{
                this.load=false;
            }
           })
            
        }
    }}

</script>