<template>
  <div>
    <!-- User List Table -->
    <div class="overflow-x-auto">
      <table class="w-full border-collapse">
        <thead class="bg-gray-50">
          <tr>
            <th class="border p-3 text-left">ID</th>
            <th class="border p-3 text-left">{{ lang.account }}</th>
            <th class="border p-3 text-left">{{ lang.user_name }}</th>
            <th class="border p-3 text-left">{{ lang.disabled }}</th>
            <th class="border p-3 text-right">
              <button 
                @click="createUser"
                class="bg-blue-500 hover:bg-blue-600 text-white px-4 py-1 rounded text-sm transition"
              >
                New
              </button>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, index) in userList" :key="row.ID" class="hover:bg-gray-50">
            <td class="border p-3">{{ row.ID }}</td>
            <td class="border p-3">{{ row.Account }}</td>
            <td class="border p-3">{{ row.Name }}</td>
            <td class="border p-3">
              <span :class="row.Disabled === 1 ? 'text-red-500' : 'text-green-500'">
                {{ row.Disabled === 1 ? lang.disabled : lang.enabled }}
              </span>
            </td>
            <td class="border p-3 text-right">
              <button 
                @click="handleEdit(index, row)"
                class="bg-gray-200 hover:bg-gray-300 px-4 py-1 rounded text-sm transition"
              >
                Edit
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Pagination -->
    <div class="flex justify-center items-center gap-2 mt-4">
      <button 
        @click="currentPage > 1 && (currentPage--, reflushList())" 
        :disabled="currentPage === 1"
        class="px-3 py-1 bg-gray-200 hover:bg-gray-300 rounded disabled:opacity-50 disabled:cursor-not-allowed"
      >
        <span class="iconify" data-icon="mdi:chevron-left"></span>
      </button>
      
      <button 
        v-for="page in visiblePages" 
        :key="page"
        @click="currentPage = page; reflushList()"
        :class="[
          'px-3 py-1 rounded',
          page === currentPage ? 'bg-blue-500 text-white' : 'bg-gray-200 hover:bg-gray-300'
        ]"
      >
        {{ page }}
      </button>
      
      <button 
        @click="currentPage < totalPage && (currentPage++, reflushList())" 
        :disabled="currentPage === totalPage"
        class="px-3 py-1 bg-gray-200 hover:bg-gray-300 rounded disabled:opacity-50 disabled:cursor-not-allowed"
      >
        <span class="iconify" data-icon="mdi:chevron-right"></span>
      </button>
    </div>

    <!-- Edit/Create User Dialog -->
    <div v-if="userInfoDialog" class="fixed inset-0 z-50 flex items-center justify-center">
      <div class="absolute inset-0 bg-black bg-opacity-50" @click="userInfoDialog = false"></div>
      <div class="relative bg-white rounded-lg p-6 w-[500px]">
        <h3 class="text-xl font-bold mb-4">{{ title }}</h3>
        
        <div class="space-y-4">
          <div>
            <label class="block text-sm font-bold mb-2">{{ lang.account }}</label>
            <input 
              v-model="editUserInfo.account"
              :disabled="editModel === 'edit'"
              class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:bg-gray-100"
            />
          </div>

          <div>
            <label class="block text-sm font-bold mb-2">{{ lang.user_name }}</label>
            <input 
              v-model="editUserInfo.name"
              class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>

          <div>
            <label class="block text-sm font-bold mb-2">{{ lang.password }}</label>
            <input 
              type="password"
              v-model="editUserInfo.password"
              :placeholder="lang.resetPwd"
              class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>

          <div class="flex items-center gap-3">
            <label class="relative inline-flex items-center cursor-pointer">
              <input 
                type="checkbox" 
                v-model="editUserInfo.disabled" 
                class="sr-only peer"
              />
              <div class="w-11 h-6 bg-gray-200 peer-focus:outline-none peer-focus:ring-4 peer-focus:ring-blue-300 rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-red-500"></div>
              <span class="ml-3 text-sm font-medium">{{ editUserInfo.disabled ? lang.disabled : lang.enabled }}</span>
            </label>
          </div>
        </div>

        <div class="flex justify-end gap-3 mt-6">
          <button 
            @click="userInfoDialog = false"
            class="px-4 py-2 bg-gray-200 hover:bg-gray-300 rounded transition"
          >
            Cancel
          </button>
          <button 
            @click="submit"
            class="px-4 py-2 bg-blue-500 hover:bg-blue-600 text-white rounded transition"
          >
            Confirm
          </button>
        </div>
      </div>
    </div>

    <!-- Toast Message -->
    <div v-if="toastMessage" class="fixed top-4 right-4 z-50">
      <div :class="[
        'px-6 py-3 rounded shadow-lg',
        toastType === 'success' ? 'bg-green-500 text-white' : 'bg-red-500 text-white'
      ]">
        <div class="font-bold">{{ toastTitle }}</div>
        <div v-if="toastMessage">{{ toastMessage }}</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import {reactive, ref, computed} from 'vue'
import lang from '../i18n/i18n';
import {http} from "@/utils/axios";

const userList = reactive([])
const currentPage = ref(1)
const totalPage = ref(1)
const userInfoDialog = ref(false)
const editModel = ref("edit")
const editUserInfo = reactive({
  "account": "",
  "name": "",
  "password": "",
  "disabled": false
})
const title = ref(lang.editUser)
const toastMessage = ref('')
const toastTitle = ref('')
const toastType = ref('success')

const visiblePages = computed(() => {
  const pages = []
  const start = Math.max(1, currentPage.value - 2)
  const end = Math.min(totalPage.value, start + 4)
  for (let i = start; i <= end; i++) {
    pages.push(i)
  }
  return pages
})

const showToast = (title, message, type = 'success') => {
  toastTitle.value = title
  toastMessage.value = message
  toastType.value = type
  setTimeout(() => {
    toastMessage.value = ''
  }, 3000)
}

const reflushList = function () {
  http.post('/api/user/list', {"current_page": currentPage.value, "page_size": 10}).then(res => {
    userList.length = 0
    totalPage.value = res.data.total_page
    userList.push(...res.data["list"])
  })
}

const handleEdit = function (idx, row) {
  editUserInfo.account = row.Account
  editUserInfo.name = row.Name
  editUserInfo.disabled = row.Disabled === 1
  editUserInfo.password = ""
  editModel.value = "edit"
  title.value = lang.editUser
  userInfoDialog.value = true
}

const createUser = function () {
  editUserInfo.account = ""
  editUserInfo.name = ""
  editUserInfo.disabled = false
  editUserInfo.password = ""
  editModel.value = "create"
  title.value = lang.newUser
  userInfoDialog.value = true
}

const submit = function () {
  if (editModel.value === 'edit') {
    let newData = {
      "account": editUserInfo.account,
      "username": editUserInfo.name,
      "disabled": editUserInfo.disabled ? 1 : 0
    }
    if (editUserInfo.password !== "") {
      newData["password"] = editUserInfo.password
    }

    http.post('/api/user/edit', newData).then(res => {
      showToast(
        res.errorNo === 0 ? lang.succ : lang.fail,
        res.errorNo === 0 ? "" : res.data,
        res.errorNo === 0 ? 'success' : 'error'
      )
      if (res.errorNo === 0) {
        reflushList()
        userInfoDialog.value = false
      }
    })
  } else {
    let newData = {
      "account": editUserInfo.account,
      "username": editUserInfo.name,
      "disabled": editUserInfo.disabled ? 1 : 0,
      "password": editUserInfo.password
    }

    http.post('/api/user/create', newData).then(res => {
      showToast(
        res.errorNo === 0 ? lang.succ : lang.fail,
        res.errorNo === 0 ? "" : res.data,
        res.errorNo === 0 ? 'success' : 'error'
      )
      if (res.errorNo === 0) {
        reflushList()
        userInfoDialog.value = false
      }
    })
  }
}

reflushList()
</script>

<style scoped>
</style>
