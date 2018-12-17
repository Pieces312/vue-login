<template>
    <div class="inner-box">
        <div class="form-header">登录</div>
        <!-- 登录表单 -->
        <form class="form-body">
            <div class="row-input" :class='{"mistakeClasses": telMsg !== ""}'>
                <input v-model='formData.account' type="text" placeholder="手机号">
                <span>{{telMsg}}</span>
            </div>
            <div class="row-input" :class='{"mistakeClasses": psdMsg !== ""}'>
                <input v-model='formData.password' type="password" placeholder="密码8~21位数字或字母">
                <span>{{psdMsg}}</span>
            </div>
            <div class="btn-row">
                <button type="button" class="btn" @click.stop='login'>登录</button>
            </div>
        </form>
        <!-- 切换注册界面 -->
        <div class="form-footer">
            <button class="btn" @click='change'>没账号？ 去注册</button>
        </div>
    </div>
</template>

<script>
import { setBase64 } from '@/utils/tools.js'
import Dialog from '@/components/loading';

export default {
    name: 'login-box',
    data() {
        return {
            // 表单数据
            formData: {
                account: '',
                password: ''
            },
            telMsg: '', //手机号错误提示
            psdMsg: '' //密码错误提示
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
        }
    },
    methods: {
        // 切换到注册页面
        change() {
            this.$store.dispatch('changeLogin', true);
        },
        // 登录事件
        login() {
            // 判断输入框内的值
            if(this.telMsg || this.psdMsg) {
                Dialog.init({
                    type: 'warn',
                    message: "您的登录信息有误"
                })
                return;
            }
            let data = Object.assign({}, this.formData);
            data.password = setBase64(data.password);

            this.$store.dispatch('login', data).then(res => {
                let self = this;
                localStorage.setItem('userInfo', JSON.stringify(res.data));
                this.psdBool = false;

                Dialog.init({
                    type: 'success',
                    message: res.msg,
                    callback: function() {
                        self.$router.replace('home')
                    }
                })
            }).catch(error => {
                Dialog.init({
                    type: 'error',
                    message: error.msg
                })
            })
        }
    }
}
</script>
