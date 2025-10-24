<template>
  <div>
    <!-- Rules Table -->
    <div class="overflow-x-auto mb-4">
      <table class="w-full border-collapse">
        <thead class="bg-gray-50">
          <tr>
            <th class="border p-3 text-left">id</th>
            <th class="border p-3 text-left">{{ lang.rule_name }}</th>
            <th class="border p-3 text-left">{{ lang.rule_do }}</th>
            <th class="border p-3 text-left">{{ lang.rule_params }}</th>
            <th class="border p-3 text-left">{{ lang.rule_priority }}</th>
            <th class="border p-3 text-left">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="row in data" :key="row.id" class="hover:bg-gray-50">
            <td class="border p-3">{{ row.id }}</td>
            <td class="border p-3">{{ row.name }}</td>
            <td class="border p-3">{{ ActionName[row.action] }}</td>
            <td class="border p-3">{{ row.params }}</td>
            <td class="border p-3">{{ row.sort }}</td>
            <td class="border p-3">
              <div class="flex items-center gap-2">
                <button 
                  @click="editRule(row)"
                  class="bg-blue-500 hover:bg-blue-600 text-white w-8 h-8 rounded-full transition"
                  title="Edit"
                >
                  <span class="iconify" data-icon="mdi:pencil"></span>
                </button>
                <button 
                  @click="confirmDelete(row.id)"
                  class="bg-red-500 hover:bg-red-600 text-white w-8 h-8 rounded-full transition"
                  title="Delete"
                >
                  <span class="iconify" data-icon="mdi:delete"></span>
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <button 
      @click="dialogVisible = true"
      class="bg-blue-500 hover:bg-blue-600 text-white px-6 py-2 rounded transition"
    >
      {{ lang.new_rule }}
    </button>

    <!-- Add/Edit Rule Dialog -->
    <div v-if="dialogVisible" class="fixed inset-0 z-50 flex items-center justify-center">
      <div class="absolute inset-0 bg-black bg-opacity-50" @click="dialogVisible = false"></div>
      <div class="relative bg-white rounded-lg p-6 w-[800px] max-h-[90vh] overflow-y-auto">
        <h3 class="text-xl font-bold mb-4">{{ lang.new_rule }}</h3>
        
        <div class="space-y-6">
          <!-- Rule Name and Priority -->
          <div class="flex gap-4">
            <div class="flex-1">
              <label class="block text-sm font-bold mb-2">{{ lang.rule_name }}</label>
              <input 
                v-model="addRuleForm.name"
                class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </div>
            <div class="w-32">
              <label class="block text-sm font-bold mb-2">{{ lang.rule_priority }}</label>
              <input 
                v-model="addRuleForm.sort"
                type="number"
                class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </div>
          </div>

          <!-- Rule Conditions -->
          <div class="border-t pt-4">
            <label class="block text-sm font-bold mb-2">{{ lang.rule_desc }}</label>
            <div class="space-y-2">
              <div v-for="(rule, index) in addRuleForm.rules" :key="index" class="flex gap-2 items-center">
                <select 
                  v-model="rule.field"
                  class="px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
                >
                  <option value="">Select field</option>
                  <option value="From">{{ lang.from }}</option>
                  <option value="Subject">{{ lang.subject }}</option>
                  <option value="To">{{ lang.to }}</option>
                  <option value="Cc">{{ lang.cc }}</option>
                  <option value="Content">{{ lang.content }}</option>
                </select>

                <select 
                  v-model="rule.type"
                  class="px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
                >
                  <option value="">Select type</option>
                  <option value="equal">{{ lang.equal }}</option>
                  <option value="contains">{{ lang.contains }}</option>
                  <option value="regex">{{ lang.regex }}</option>
                </select>

                <input 
                  v-model="rule.rule"
                  class="flex-1 px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
                  placeholder="Value"
                />

                <button 
                  @click="removeRuleLine(index)"
                  class="bg-red-500 hover:bg-red-600 text-white w-8 h-8 rounded-full transition"
                >
                  <span class="iconify" data-icon="mdi:minus"></span>
                </button>
              </div>
            </div>
            <button 
              @click="addRule"
              class="mt-2 bg-blue-500 hover:bg-blue-600 text-white w-8 h-8 rounded-full transition"
            >
              <span class="iconify" data-icon="mdi:plus"></span>
            </button>
          </div>

          <!-- Rule Action -->
          <div class="border-t pt-4">
            <label class="block text-sm font-bold mb-2">{{ lang.rule_do }}</label>
            <div class="flex gap-2">
              <select 
                v-model="addRuleForm.action"
                @change="ruleTypeChange"
                class="px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              >
                <option value="">Select action</option>
                <option :value="READ">{{ lang.mark_read }}</option>
                <option :value="MOVE">{{ lang.move }}</option>
                <option :value="DELETE">{{ lang.delete }}</option>
                <option :value="FORWARD">{{ lang.forward }}</option>
              </select>

              <select 
                v-if="addRuleForm.action === 4"
                v-model="addRuleForm.params"
                @click="reflushGroupInfos"
                class="flex-1 px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              >
                <option value="">Select group</option>
                <option v-for="gp in groupData.list" :key="gp.id" :value="gp.id">{{ gp.name }}</option>
              </select>

              <input 
                v-if="addRuleForm.action === 2"
                v-model="addRuleForm.params"
                placeholder="Forward Email Address"
                class="flex-1 px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </div>
          </div>
        </div>

        <div class="flex justify-end gap-3 mt-6">
          <button 
            @click="dialogVisible = false"
            class="px-4 py-2 bg-gray-200 hover:bg-gray-300 rounded transition"
          >
            Cancel
          </button>
          <button 
            @click="submitRule"
            class="px-4 py-2 bg-blue-500 hover:bg-blue-600 text-white rounded transition"
          >
            {{ lang.submit }}
          </button>
        </div>
      </div>
    </div>

    <!-- Confirm Delete Dialog -->
    <div v-if="showConfirmDelete" class="fixed inset-0 z-50 flex items-center justify-center">
      <div class="absolute inset-0 bg-black bg-opacity-50" @click="showConfirmDelete = false"></div>
      <div class="relative bg-white rounded-lg p-6 w-96">
        <h3 class="text-lg font-bold mb-4">{{ lang.del_rule_confirm }}</h3>
        <div class="flex justify-end gap-3">
          <button 
            @click="showConfirmDelete = false"
            class="px-4 py-2 bg-gray-200 hover:bg-gray-300 rounded"
          >
            No, Thanks
          </button>
          <button 
            @click="delRule(deleteId)"
            class="px-4 py-2 bg-red-500 hover:bg-red-600 text-white rounded"
          >
            Yes
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
import {reactive, ref} from 'vue';
import lang from '../i18n/i18n';
import {http} from "@/utils/axios";

const data = ref([])
const dialogVisible = ref(false)
const showConfirmDelete = ref(false)
const deleteId = ref(0)
const toastMessage = ref('')
const toastTitle = ref('')
const toastType = ref('success')

const READ = 1
const FORWARD = 2
const DELETE = 3
const MOVE = 4

const ActionName = {
  1: lang.mark_read,
  2: lang.forward,
  3: lang.delete,
  4: lang.move
}

const showToast = (title, message, type = 'success') => {
  toastTitle.value = title
  toastMessage.value = message
  toastType.value = type
  setTimeout(() => {
    toastMessage.value = ''
  }, 3000)
}

const init = function () {
  http.post("/api/rule/get").then((res) => {
    data.value = res.data
  })
}

init()

const groupData = reactive({
  list: []
})

const reflushGroupInfos = function () {
  http.get('/api/group/list').then(res => {
    if (res.data != null) {
      groupData.list = res.data
      for (let i = 0; i < groupData.list.length; i++) {
        groupData.list[i].id += ""
      }
    }
  })
}

reflushGroupInfos()

const addRuleForm = reactive({
  "id": 0,
  "name": "",
  "sort": 0,
  "rules": [
    {
      "field": "",
      "type": "",
      "rule": ""
    }
  ],
  "action": "",
  "params": ""
})

const confirmDelete = function (id) {
  deleteId.value = id
  showConfirmDelete.value = true
}

const delRule = function (id) {
  http.post("/api/rule/del", {"id": id}).then((res) => {
    showToast(
      res.errorNo === 0 ? lang.succ : lang.fail,
      res.data,
      res.errorNo === 0 ? 'success' : 'error'
    )
    showConfirmDelete.value = false
    init()
  })
}

const editRule = function (ruleInfo) {
  addRuleForm.id = ruleInfo.id
  addRuleForm.name = ruleInfo.name
  addRuleForm.rules = ruleInfo.rules
  addRuleForm.action = ruleInfo.action
  addRuleForm.params = ruleInfo.params
  addRuleForm.sort = ruleInfo.sort
  dialogVisible.value = true
}

const removeRuleLine = function (index) {
  addRuleForm.rules.splice(index, 1);
}

const addRule = function () {
  addRuleForm.rules.push({
    "field": "",
    "type": "",
    "rule": ""
  })
}

const submitRule = function () {
  let api = "/api/rule/add"
  if (addRuleForm.id > 0) {
    api = "/api/rule/update"
  }

  addRuleForm.sort = parseInt(addRuleForm.sort)

  http.post(api, addRuleForm).then((res) => {
    if (res.errorNo !== 0) {
      showToast(lang.fail, res.data, 'error')
    } else {
      init()
      dialogVisible.value = false

      addRuleForm.id = 0
      addRuleForm.name = ""
      addRuleForm.sort = 0
      addRuleForm.rules = [
        {
          "field": "",
          "type": "",
          "rule": ""
        }
      ]
      addRuleForm.action = ""
      addRuleForm.params = ""
      showToast(lang.succ, '', 'success')
    }
  })
}

const ruleTypeChange = function () {
  addRuleForm.params = ''
}
</script>

<style scoped>
</style>
