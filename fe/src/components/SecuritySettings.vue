<template>
  <div class="max-w-2xl">
    <div class="border-b border-gray-300 mb-6 pb-2">
      <h3 class="text-lg font-bold">{{ lang.modify_pwd }}</h3>
    </div>

    <div class="mb-4">
      <label class="block text-sm font-bold mb-2">{{ lang.modify_pwd }}</label>
      <input 
        type="password" 
        v-model="ruleForm.new_pwd"
        class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <span v-if="errors.new_pwd" class="text-red-500 text-sm">{{ errors.new_pwd }}</span>
    </div>

    <div class="mb-6">
      <label class="block text-sm font-bold mb-2">{{ lang.enter_again }}</label>
      <input 
        type="password" 
        v-model="ruleForm.new_pwd2"
        class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <span v-if="errors.new_pwd2" class="text-red-500 text-sm">{{ errors.new_pwd2 }}</span>
    </div>

    <button 
      @click="submit"
      class="bg-blue-500 hover:bg-blue-600 text-white px-6 py-2 rounded transition mb-8"
    >
      {{ lang.submit }}
    </button>

    <div class="border-b border-gray-300 mb-6 pb-2">
      <h3 class="text-lg font-bold">{{ lang.logout }}</h3>
    </div>

    <button 
      @click="logout"
      class="bg-red-500 hover:bg-red-600 text-white px-6 py-2 rounded transition"
    >
      <span class="iconify mr-2" data-icon="mdi:logout"></span>{{ lang.logout }}
    </button>

    <!-- Toast Message -->
    <div v-if="toastMessage" class="fixed top-4 right-4 z-50">
      <div :class="[
        'px-6 py-3 rounded shadow-lg',
        toastType === 'success' ? 'bg-green-500 text-white' : 'bg-red-500 text-white'
      ]">
        <div class="font-bold">{{ toastTitle }}</div>
        <div>{{ toastMessage }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import {reactive, ref} from 'vue'
import lang from '../i18n/i18n';
import {http} from "@/utils/axios";

const ruleForm = reactive({
  new_pwd: "",
  new_pwd2: ""
})

const errors = reactive({
  new_pwd: '',
  new_pwd2: ''
})

const toastMessage = ref('')
const toastTitle = ref('')
const toastType = ref('success')

const showToast = (title, message, type = 'success') => {
  toastTitle.value = title
  toastMessage.value = message
  toastType.value = type
  setTimeout(() => {
    toastMessage.value = ''
  }, 3000)
}

const logout = function () {
  http.post("/api/logout", {}).then(() => {
    location.reload();
  })
}

const submit = function () {
  errors.new_pwd = ''
  errors.new_pwd2 = ''

  if (ruleForm.new_pwd === "") {
    errors.new_pwd = lang.err_required_pwd
    return
  }

  if (ruleForm.new_pwd2 === "") {
    errors.new_pwd2 = lang.err_required_pwd
    return
  }

  if (ruleForm.new_pwd !== ruleForm.new_pwd2) {
    showToast('Error', lang.err_pwd_diff, 'error')
    return
  }

  http.post("/api/settings/modify_password", {password: ruleForm.new_pwd}).then(res => {
    showToast(
      res.errorNo === 0 ? lang.succ : lang.fail,
      res.data,
      res.errorNo === 0 ? 'success' : 'error'
    )
    if (res.errorNo === 0) {
      ruleForm.new_pwd = ''
      ruleForm.new_pwd2 = ''
    }
  })
}
</script>

<style scoped>
</style>
