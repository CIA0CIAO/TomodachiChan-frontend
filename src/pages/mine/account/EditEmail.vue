<script lang="ts" setup>
import SendCode from '@/components/SendCode.vue'
import { computed, reactive } from 'vue'
import { useRouter } from 'vue-router'
import {updateEmail} from "@/api/user.ts";

const router = useRouter()

const email = reactive({ value: '', valid: false })
const code = reactive({ value: '', valid: false })

const handleCodeChange = (value: string) => {
  code.value = value
  code.valid = /^\d{6}$/.test(value)
}

const updateButtonValid = computed(() => {
  return email.valid && code.valid
})

const handleSubmit = () => {
  updateEmail(email.value, code.value).then(() => {
    localStorage.removeItem('userId')
    localStorage.removeItem('token')
    router.push('/login')
  })
}
</script>

<template>
  <van-form label-width="50px" @submit="handleSubmit">
    <van-cell-group inset>
      <!-- 邮箱 -->
      <van-field
          v-model="email.value"
          :update:model-value="email.valid = /^[A-Za-z0-9+_.-]+@(.+)$/.test(email.value)"
          label="邮箱"
          placeholder="请输入邮箱"
          style="margin-top: 16px"
          type="email"
      />
      <!-- 验证码 -->
      <SendCode
          :email="email.value"
          :emailValid="email.valid"
          @update:code="handleCodeChange"
      />
      <!-- 换绑按钮 -->
      <van-button
          :disabled="!updateButtonValid"
          block
          native-type="submit"
          style="margin-top: 16px"
          type="primary"
      >
        换绑
      </van-button>
    </van-cell-group>
  </van-form>
</template>
