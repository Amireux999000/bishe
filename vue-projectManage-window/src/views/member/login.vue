<template>
    <div class="main">
        <a-form
                class="user-layout-login"
                ref="formLogin"
                id="formLogin"
                :form="form"
        >
            <a-tabs
                    :activeKey="customActiveKey"
                    :tabBarStyle="{ textAlign: 'center', borderBottom: 'unset' }"
                    @change="handleTabClick"
            >
                <a-tab-pane key="tab1" tab="账号密码登录">
                    <a-form-item>
                        <a-input size="large" type="text" placeholder="帐户名或邮箱地址 / 123456"
                                 v-decorator="[
                                'account',
                                {rules: [{ required: true, message: '请输入帐户名或邮箱地址' },{ validator: this.handleUsernameOrEmail }], validateTrigger: 'blur'}
                            ]">
                            <a-icon slot="prefix" type="user" :style="{ color: 'rgba(0,0,0,.25)' }"/>
                        </a-input>
                    </a-form-item>

                    <a-form-item
                    >
                        <a-input size="large" type="password" autocomplete="false" placeholder="密码 / 123456"
                                 v-decorator="[
                                'password',
                                {rules: [{ required: true, message: '请输入密码' }], validateTrigger: 'blur'}
                            ]">
                            <a-icon slot="prefix" type="lock" :style="{ color: 'rgba(0,0,0,.25)' }"/>
                        </a-input>
                    </a-form-item>
                </a-tab-pane>
                <a-tab-pane key="tab2" tab="手机号登录">
                    <a-form-item
                    >
                        <a-input size="large" type="text" placeholder="手机号"
                                 v-decorator="[
                                'mobile',
                                {rules: [{ required: true, pattern: /^1[34578]\d{9}$/, message: '请输入正确的手机号' },{ validator: this.handleUsernameOrEmail }], validateTrigger: 'change'}
                            ]">
                            <a-icon slot="prefix" type="mobile" :style="{ color: 'rgba(0,0,0,.25)' }"/>
                        </a-input>
                    </a-form-item>

                    <a-row :gutter="16">
                        <a-col class="gutter-row" :span="16">
                            <a-form-item
                            >
                                <a-input size="large" type="text" placeholder="验证码"
                                         v-decorator="[
                                'captcha',
                                {rules: [{ required: true, message: '请输入验证码' }], validateTrigger: 'blur'}
                            ]">
                                    <a-icon slot="prefix" type="mail" :style="{ color: 'rgba(0,0,0,.25)' }"/>
                                </a-input>
                            </a-form-item>
                        </a-col>
                        <a-col class="gutter-row" :span="8">
                            <a-button
                                    class="getCaptcha"
                                    tabindex="-1"
                                    :disabled="state.smsSendBtn"
                                    @click.stop.prevent="getCaptcha"
                                    v-text="!state.smsSendBtn && '获取验证码' || (state.time+' s')"
                            ></a-button>
                        </a-col>
                    </a-row>
                </a-tab-pane>
            </a-tabs>

            <a-form-item>
                <a-checkbox v-model="formLogin.rememberMe">自动登录</a-checkbox>
                <a class="forge-password" style="float: right;">
                    <router-link class="register" :to="{ name: 'forget' }">忘记密码</router-link>
                </a>
            </a-form-item>

            <a-form-item style="margin-top:24px">
                <a-button
                        size="large"
                        type="primary"
                        htmlType="submit"
                        class="login-button"
                        :loading="loginBtn"
                        @click.stop.prevent="handleSubmit"
                        :disabled="loginBtn"
                >登录
                </a-button>
            </a-form-item>

            <div class="user-login-other">
                <span>其他登录方式</span>
                <a v-on:click="qqLogin()">
                    <a-icon class="item-icon" type="qq"></a-icon>
                </a>
                <a v-on:click="sinaLogin()">
                    <a-icon class="item-icon" type="weibo-circle"></a-icon>
                </a>
             <!--   <router-link class="register" :to="{ name: 'register' }">注册账户</router-link>-->
            </div>
        </a-form>
          <Socket ref="socket" v-if="config.WS_URI"></Socket>
        <a-drawer
                title="QQ登录"
                :width="720"
                @close="onClose"
                :visible="visible"
                :wrapStyle="{height: 'calc(100% - 108px)',overflow: 'auto',paddingBottom: '108px'}"
        >
            <div>
                <iframe
                        :src="currentSrc"
                        frameborder="0"
                        scrolling=""
                        style="background-color:transparent; position: absolute; z-index: -1; width: 100%; height: 100%; top: 55px;left:0; scrolling=auto">
                </iframe>
            </div>
        </a-drawer>
    </div>
</template>

<script>
    import {mapActions} from 'vuex'
    import {getCaptcha, Login, getQqLoginUrl, qqCallback, getSinaLoginUrl} from '@/api/mock'
    import {info} from '@/api/system';
    import {checkResponse, createRoute, timeFix} from '@/assets/js/utils'
    import {getStore, setStore} from '@/assets/js/storage';
    import Socket from '../../components/websocket/socket';
    import config from "../../config/config";

    export default {
        components: {
            Socket
        },
        data() {
            return {
                customActiveKey: 'tab1',
                loginBtn: false,
                // login type: 0 email, 1 account, 2 telephone
                loginType: 0,
                requiredTwoStepCaptcha: false,
                stepCaptchaVisible: false,
                form: this.$form.createForm(this),
                state: {
                    time: 60,
                    smsSendBtn: false
                },
                formLogin: {
                    account: '',
                    password: '',
                    captcha: '',
                    mobile: '',
                    rememberMe: true
                },
                config: config,
                visible: false,
                currentSrc: "http://www.baidu.com/",
            }
        },
        computed: {
        },
        created() {
        },
        methods: {
            ...mapActions(['Login', 'Logout']),
            // handler
            handleUsernameOrEmail(rule, value, callback) {
                const regex = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+((\.[a-zA-Z0-9_-]{2,3}){1,2})$/;
                if (regex.test(value)) {
                    this.loginType = 0
                } else {
                    this.loginType = 1
                }
                callback()
            },
            qqLogin(){
                getQqLoginUrl().then(res => {
                    if (checkResponse(res, true)) {
                        this.currentSrc = res.url;
                        this.visible = true;
                    }
                }).catch(res => {
                });
            },
            sinaLogin(){
                getSinaLoginUrl().then(res => {
                    if (checkResponse(res, true)) {
                        window.location.href = res.url;
                    }
                }).catch(res => {
                });
            },
            handleTabClick(key) {
                this.customActiveKey = key
                // this.form.resetFields()
            },
            handleSubmit() {
                const app = this;
                let flag = false;

                let loginParams = {
                    remember_me: app.formLogin.rememberMe
                };

                // 使用账户密码登录
                if (app.customActiveKey === 'tab1') {
                    app.form.validateFields(['account', 'password'], {force: true}, (err, values) => {
                        if (!err) {
                            flag = true;
                            loginParams[!app.loginType ? 'account' : 'account'] = values.account;
                            //loginParams.password = md5(values.password)
                            loginParams.password = values.password;
                            loginParams.type = '0';
                        }
                    })
                    // 使用手机号登录
                } else {
                    app.form.validateFields(['mobile', 'captcha'], {force: true}, (err, values) => {
                        if (!err) {
                            flag = true;
                            loginParams = Object.assign(loginParams, values)
                            loginParams.type = '1';
                        }
                    })
                }

                if (!flag) return;
                app.loginBtn = true;
                console.log(loginParams)
                //loginParams.clientid = getStore('client_id');
                Login(loginParams).then(res => {
                    if (checkResponse(res, true)) {
                        loginParams.token = res.token;
                        setStore("token", res.token);
                        const obj = {
                            userInfo: res.data.member,
                            tokenList: 7033929,
                        };
                        app.$store.dispatch('SET_LOGGED', obj);
                        app.$store.dispatch('GET_MENU').then(() => {
                            app.loginBtn = false;
                            app.loginSuccess(res);
                        });
                    }
                    this.loginBtn = false
                }).catch(res => {
                    this.loginBtn = false
                });
            },
            getCaptcha(e) {
                e.preventDefault();
                const app = this;

                this.form.validateFields(['mobile'], {force: true}, (err, values) => {
                    if (!err) {
                        this.state.smsSendBtn = true;

                        const interval = window.setInterval(() => {
                            if (app.state.time-- <= 0) {
                                app.state.time = 60;
                                app.state.smsSendBtn = false;
                                window.clearInterval(interval)
                            }
                        }, 1000);

                        const hide = this.$message.loading('验证码发送中..', 0);
                        getCaptcha(values.mobile)
                            .then(res => {
                                this.$message.destroy();
                                if (!checkResponse(res)) {
                                    return false;
                                }
                                let tips = '验证码获取成功';
                                if (res.data) {
                                    tips += '，您的验证码为：' + res.data;
                                }
                                this.$notification['success']({
                                    message: '提示',
                                    description: tips,
                                    duration: 8,
                                    placement: 'bottomLeft'
                                });
                            })
                            .catch(err => {
                                // setTimeout(hide, 1);
                                clearInterval(interval);
                                app.state.time = 60;
                                app.state.smsSendBtn = false;
                                this.requestFailed(err)
                            })
                    }
                })
            },
            loginSuccess(res) {
                setTimeout(() => {
                    const menu = getStore('menu', true);
                    if (menu) {
                        let routes = this.$router.options.routes;
                        menu.forEach(function (v) {
                            routes[0].children.push(createRoute(v));
                            if (v.children) {
                                v.children.forEach(function (v2) {
                                    routes[0].children.push(createRoute(v2));
                                    if (v2.children) {
                                        v2.children.forEach(function (v3) {
                                            routes[0].children.push(createRoute(v3));
                                        });
                                    }
                                });
                            }
                        });
                        this.loginBtn = false;
                        this.$router.addRoutes(routes);
                        let redirect = this.$route.query.redirect || config.HOME_PAGE;
                        this.$router.push({
                            path: redirect
                        });
                        this.$notification.success({
                            message: '欢迎',
                            description: `${res.data.member.username}，${timeFix()}，欢迎回来`,
                        })
                    }
                }, 500);
            },
            requestFailed(err) {
                this.$notification['error']({
                    message: '错误',
                    description: ((err.response || {}).data || {}).message || '请求出现错误，请稍后再试',
                    duration: 4
                });
                this.loginBtn = false
            },
            onClose() {
                this.visible = false
            },
        }
    }
</script>

<style lang="less">
    .user-layout-login {
        label {
            font-size: 14px;
        }

        .getCaptcha {
            display: block;
            width: 100%;
            height: 40px;
        }

        .forge-password {
            font-size: 14px;
        }

        button.login-button {
            padding: 0 15px;
            font-size: 16px;
            height: 40px;
            width: 100%;
        }

        .user-login-other {
            text-align: left;
            margin-top: 24px;
            line-height: 22px;

            .item-icon {
                font-size: 24px;
                color: rgba(0, 0, 0, 0.2);
                margin-left: 16px;
                vertical-align: middle;
                cursor: pointer;
                transition: color 0.3s;

                &:hover {
                    color: #1890ff;
                }
            }

            .register {
                float: right;
            }
        }
    }
</style>
