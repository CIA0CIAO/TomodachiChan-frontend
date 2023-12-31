<script lang="ts" setup>
import { loginByPassword, loginByVerificationCode } from '@/api/user.ts'
import SendCode from '@/components/SendCode.vue'
import { computed, reactive, ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

/**
 * 登录表单项
 */
const email = reactive({ value: '', valid: false })
const password = reactive({ value: '', valid: false })
const code = reactive({ value: '', valid: false })

const handleCodeChange = (value: string) => {
    code.value = value
    code.valid = /^\d{6}$/.test(value)
}

/**
 * 登录按钮
 */
const loginByCode = ref(true)

const loginButtonValid = computed(() => {
    return loginByCode.value ?
        email.valid && code.valid :
        email.valid && password.valid
})

const handleSubmit = () => {
    let operation = loginByCode.value ?
        loginByVerificationCode(email.value, code.value) :
        loginByPassword(email.value, password.value)
    operation.then((resp) => {
        let { user, token } = resp.data.data
        localStorage.setItem('userId', user.id)
        localStorage.setItem('token', token)
        router.back()
    })
}
</script>

<template>
    <!-- Logo -->
    <van-row justify="center" style="margin: 32px">
        <van-image
            src="public/ani-crazy.png"
            width="40%"
        />
    </van-row>

    <!-- 登录表单 -->
    <van-form label-width="50px" @submit="handleSubmit">
        <van-cell-group inset>
            <!-- 邮箱 -->
            <van-field
                v-model="email.value"
                :update:model-value="email.valid = /^[A-Za-z0-9+_.-]+@(.+)$/.test(email.value)"
                label="邮箱"
                placeholder="请输入邮箱"
                type="email"
            />
            <!-- 验证码 -->
            <SendCode
                v-if="loginByCode"
                :email="email.value"
                :emailValid="email.valid"
                @update:code="handleCodeChange"
            />
            <!-- 密码 -->
            <van-field
                v-if="!loginByCode"
                v-model="password.value"
                :update:model-value="password.valid = /^.{6,}$/.test(password.value)"
                label="密码"
                placeholder="请输入密码"
                type="password"
            />
        </van-cell-group>

        <van-row justify="center" style="margin-top: 16px">
            <!-- 登录方式 -->
            <van-col span="8">
                <van-button block plain type="default" @click="loginByCode = !loginByCode">
                    {{ loginByCode ? '密码登录' : '验证码登录' }}
                </van-button>
            </van-col>
            <!-- 登录按钮 -->
            <van-col offset="1" span="12">
                <van-button :disabled="!loginButtonValid" block native-type="submit" type="primary">
                    {{ loginByCode ? '登录 / 注册' : '登录' }}
                </van-button>
            </van-col>
        </van-row>
    </van-form>
</template>
