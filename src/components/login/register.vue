<template>
    <div class="inner-box register" :loading='showLoading'>
        <div class="form-header">注册</div>
        <form class="form-body" id='formData'>
            <div class="row-input" :class='{"mistakeClasses": telMsg.length}'>
                <input v-model='formData.account' type="text" placeholder="手机号">
                <span>{{telMsg}}</span>
            </div>
            <div class="row-input" :class='{"mistakeClasses": psdMsg.length}'>
                <input v-model='formData.password' type="password" placeholder="密码8~21位数字或字母">
                <span>{{psdMsg}}</span>
            </div>
            <div class="row-input" :class='{"mistakeClasses": repsdMsg.length}'>
                <input v-model='formData.repassword' type="password" placeholder="再次确认密码">
                <span>{{repsdMsg}}</span>
            </div>
            <div class="btn-row">
                <button type="button" class="btn" @click.stop='signUpAccount'>保存</button>
            </div>
        </form>
        <div class="form-footer">
            <button class="btn" @click='change'>已有账号</button>
        </div>
    </div>
</template>

<script>
import { setBase64 } from '@/utils/tools.js'
import {Ajax} from '@/api/index.js';
import Dialog from '@/components/loading';

export default {
    name: 'register-box',
    data() {
        return {
            formData: {
                account: '',
                password: '',
                repassword: ''
            },
            showLoading: false,
            telMsg: '', //手机号错误提示
            psdMsg: '', //密码错误提示
            repsdMsg: '', //确认密码错误提示
        }
    },
    watch: {
       //判断手机号是否输入正确
        "formData.account"(val) {
            let reg = /^[1][3,5,7,8,9][0-9]{9}$/;
            let res = (val === "") ? '账号不能为空' : (!reg.test(val) ? "输入账号有误" : "");

            this.telMsg = res
        },
        //判断密码是否输入正确
        "formData.password"(val) {
            let reg = /^[A-Za-z0-9]{8,21}$/;
            let res = (val === "") ? '密码不能为空' : (!reg.test(val) ? "密码格式错误" : "");
            
            this.psdMsg = res
        },
        //判断两次密码是否输入一致
        "formData.repassword"(val) {
            let reg = this.formData.password;
            let res = (val === "") ? '确认密码不能为空' : ((reg !== val) ? "两次密码输入不一致" : "");
            
            this.repsdMsg = res;
        }
    },
    methods: {
        // 切换到登录页面
        change() {
            this.$store.dispatch('changeLogin', false);
        },
        // 注册账号按钮
        signUpAccount() {
            // 判断输入框内的值
            if(this.telMsg || this.psdMsg || this.repsdMsg) {
                Dialog.init({
                    type: 'warn',
                    message: "注册信息有误"
                })
                return;
            }
            this.showLoading = true;

            let data = Object.assign({}, this.formData);
            data.password = setBase64(data.password);
            data.repassword = setBase64(data.repassword);

            this.$store.dispatch("signUpUserInfo", data).then(res => {
                let self = this;
                this.showLoading = false;

                Dialog.init({
                    type: 'success',
                    message: res.msg,
                    callback: function() {
                        self.$store.dispatch('changeLogin', false);
                    }
                })

            }).catch(error => {
                this.showLoading = false;

                Dialog.init({
                    type: 'error',
                    message: error.msg
                })
            })
        }
    }
}
</script>
