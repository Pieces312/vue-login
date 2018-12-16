<template>
    <div class="inner-box">
        <div class="form-header">登录</div>
        <!-- 登录表单 -->
        <form class="form-body">
            <div class="row-input" :class='{"mistakeClasses": telMsg.length}'>
                <input v-model='formData.account' type="text" placeholder="手机号">
                <span>{{telMsg}}</span>
            </div>
            <div class="row-input" :class='{"mistakeClasses": psdMsg.length}'>
                <input v-model='formData.password' type="password" placeholder="密码">
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
            }
        }
    },
    computed: {
      //判断手机号是否输入正确
      telMsg() {
        if(!this.formData.account) {
          return '账号不能为空'
        }

        let reg = /^[1][3,5,7,8,9][0-9]{9}$/;
        let isTrue = reg.test(this.formData.account);

        if(!isTrue) {
          return "输入账号有误"
        }

        return ""
      },
      //判断密码是否输入正确
      psdMsg() {
        if(!this.formData.password) {
          return '密码不能为空'
        }

        let reg = /^[A-Za-z0-9]{8,21}$/;
        let isTrue = reg.test(this.formData.password);

        if(!isTrue) {
          return "密码格式错误"
        }

        return ""

      }
    },
    methods: {
        // 切换到注册页面
        change() {
            this.$store.dispatch('changeLogin', true);
        },
        // 登录事件
        login() {
            let {account, password} = this.formData;

            if(!account || !password) {
                this.telMsg = !account ? true : false;
                this.psdMsg = !password ? true : false;
                return;
            }

            // 再次判断输入框内的值
            if(this.telMsg || this.psdMsg) {
                Dialog.init({
                    type: 'warn',
                    message: "您的登录信息有误"
                })
                return;
            }

            this.formData.password = setBase64(password)

            this.$store.dispatch('login', this.formData).then(res => {
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
