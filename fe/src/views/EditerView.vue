<template>
  <div class="text-left pr-5 py-4 px-5">
    <form @submit.prevent="send">
      <!-- Sender Field -->
      <div class="mb-4">
        <label class="block text-sm font-bold mb-2">{{ lang.sender }}</label>
        <div class="relative">
          <div 
            @click="showSenderPopover = !showSenderPopover"
            class="border border-gray-300 rounded px-3 py-2 cursor-pointer hover:border-gray-400 transition"
          >
            <span class="font-bold">{{ ruleForm.nickName }}</span>
            <span> &lt;{{ ruleForm.sender }}@{{ ruleForm.pickDomain }}&gt;</span>
          </div>
          
          <!-- Sender Popover -->
          <div v-if="showSenderPopover" class="absolute z-10 mt-2 w-full bg-white border rounded-lg shadow-lg p-4">
            <div class="mb-4">
              <label class="block text-sm font-bold mb-2">{{ lang.sender }}</label>
              <div class="flex items-center gap-2">
                <input 
                  v-model="ruleForm.sender" 
                  :disabled="!(globalStatus.userInfos.is_admin)"
                  :placeholder="lang.sender_desc"
                  class="flex-1 px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
                />
                <span>@</span>
                <select 
                  v-model="ruleForm.pickDomain"
                  class="px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
                >
                  <option v-for="item in ruleForm.domains" :key="item" :value="item">{{ item }}</option>
                </select>
              </div>
            </div>
            
            <div>
              <label class="block text-sm font-bold mb-2">{{ lang.nick_name }}</label>
              <input 
                v-model="ruleForm.nickName"
                class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </div>
            
            <button 
              @click="showSenderPopover = false"
              type="button"
              class="mt-3 w-full bg-blue-500 hover:bg-blue-600 text-white py-2 rounded"
            >
              {{ lang.confirm || 'OK' }}
            </button>
          </div>
        </div>
        <span v-if="errors.sender" class="text-red-500 text-sm">{{ errors.sender }}</span>
      </div>

      <!-- To Field -->
      <div class="mb-4">
        <label class="block text-sm font-bold mb-2">{{ lang.to }}</label>
        <div class="border rounded p-2 min-h-[40px] flex flex-wrap gap-2">
          <span 
            v-for="(receiver, index) in ruleForm.receivers" 
            :key="index"
            class="inline-flex items-center bg-blue-100 text-blue-800 px-2 py-1 rounded"
          >
            {{ receiver }}
            <button @click="removeReceiver(index)" type="button" class="ml-1">
              <span class="iconify" data-icon="mdi:close"></span>
            </button>
          </span>
          <input 
            v-model="receiverInput"
            @keydown.enter.prevent="addReceiver"
            @keydown.comma.prevent="addReceiver"
            :placeholder="lang.to_desc"
            class="flex-1 min-w-[200px] outline-none"
          />
        </div>
        <span v-if="errors.receivers" class="text-red-500 text-sm">{{ errors.receivers }}</span>
      </div>

      <!-- CC Field -->
      <div class="mb-4">
        <label class="block text-sm font-bold mb-2">{{ lang.cc }}</label>
        <div class="border rounded p-2 min-h-[40px] flex flex-wrap gap-2">
          <span 
            v-for="(cc, index) in ruleForm.cc" 
            :key="index"
            class="inline-flex items-center bg-blue-100 text-blue-800 px-2 py-1 rounded"
          >
            {{ cc }}
            <button @click="removeCC(index)" type="button" class="ml-1">
              <span class="iconify" data-icon="mdi:close"></span>
            </button>
          </span>
          <input 
            v-model="ccInput"
            @keydown.enter.prevent="addCC"
            @keydown.comma.prevent="addCC"
            :placeholder="lang.cc_desc"
            class="flex-1 min-w-[200px] outline-none"
          />
        </div>
        <span v-if="errors.cc" class="text-red-500 text-sm">{{ errors.cc }}</span>
      </div>

      <!-- BCC Field -->
      <div class="mb-4">
        <label class="block text-sm font-bold mb-2">{{ lang.bcc }}</label>
        <div class="border rounded p-2 min-h-[40px] flex flex-wrap gap-2">
          <span 
            v-for="(bcc, index) in ruleForm.bcc" 
            :key="index"
            class="inline-flex items-center bg-blue-100 text-blue-800 px-2 py-1 rounded"
          >
            {{ bcc }}
            <button @click="removeBCC(index)" type="button" class="ml-1">
              <span class="iconify" data-icon="mdi:close"></span>
            </button>
          </span>
          <input 
            v-model="bccInput"
            @keydown.enter.prevent="addBCC"
            @keydown.comma.prevent="addBCC"
            :placeholder="lang.bcc_desc"
            class="flex-1 min-w-[200px] outline-none"
          />
        </div>
        <span v-if="errors.bcc" class="text-red-500 text-sm">{{ errors.bcc }}</span>
      </div>

      <!-- Subject Field -->
      <div class="mb-4">
        <label class="block text-sm font-bold mb-2">{{ lang.title }}</label>
        <input 
          v-model="ruleForm.subject"
          :placeholder="lang.title"
          class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
        />
        <span v-if="errors.subject" class="text-red-500 text-sm">{{ errors.subject }}</span>
      </div>

      <!-- Editor -->
      <div class="mb-4 border rounded">
        <Toolbar class="border-b" :editor="editorRef" :defaultConfig="toolbarConfig" :mode="mode"/>
        <Editor style="height: 300px;" v-model="valueHtml" :defaultConfig="editorConfig" :mode="mode" @onCreated="handleCreated"/>
      </div>

      <!-- File List -->
      <div v-if="fileList.length > 0" class="mb-4">
        <ul class="list-none p-0">
          <li 
            v-for="(item, index) in fileList" 
            :key="index"
            class="flex items-center justify-between bg-gray-100 px-3 py-2 mb-2 rounded"
          >
            <span>{{ item.name }}</span>
            <button @click="delFile(index)" type="button" class="text-red-500 hover:text-red-700">
              <span class="iconify" data-icon="mdi:close"></span>
            </button>
          </li>
        </ul>
      </div>

      <!-- Action Buttons -->
      <div class="flex gap-3">
        <button 
          type="submit"
          class="bg-blue-500 hover:bg-blue-600 text-white px-6 py-2 rounded transition"
        >
          <span class="iconify mr-2" data-icon="mdi:send"></span>{{ lang.send }}
        </button>
        
        <button 
          @click="upload" 
          type="button"
          class="bg-gray-200 hover:bg-gray-300 px-6 py-2 rounded transition"
        >
          <span class="iconify mr-2" data-icon="mdi:attachment"></span>{{ lang.add_att }}
        </button>
        <input v-show="false" ref="fileRef" type="file" @change="fileChange">
      </div>
    </form>

    <!-- Toast Message -->
    <div v-if="toastMessage" class="fixed top-4 right-4 z-50">
      <div :class="[
        'px-6 py-3 rounded shadow-lg',
        toastType === 'success' ? 'bg-green-500 text-white' : 'bg-red-500 text-white'
      ]">
        {{ toastMessage }}
      </div>
    </div>
  </div>
</template>

<script setup>
import '@wangeditor/editor/dist/css/style.css'
import {onBeforeUnmount, reactive, ref, shallowRef} from 'vue'
import lang from '../i18n/i18n';
import {Editor, Toolbar} from '@wangeditor/editor-for-vue'
import {i18nChangeLanguage} from '@wangeditor/editor'
import {useRouter} from 'vue-router';
import {http} from "@/utils/axios";
import useGroupStore from '../stores/group'
import {useGlobalStatusStore} from "@/stores/useGlobalStatusStore";

const router = useRouter();
const groupStore = useGroupStore()
const globalStatus = useGlobalStatusStore();

if (lang.lang === "zhCn") {
  i18nChangeLanguage('zh-CN')
} else {
  i18nChangeLanguage('en')
}

const valueHtml = ref('<p>hello</p>')
const toolbarConfig = {}
const editorConfig = {
  MENU_CONF: {},
  placeholder: ''
}

editorConfig.MENU_CONF['uploadImage'] = {
  base64LimitSize: 100 * 1024 * 1024 * 1024,
}

const mode = ref()
const fileRef = ref();
const showSenderPopover = ref(false)
const receiverInput = ref('')
const ccInput = ref('')
const bccInput = ref('')
const errors = reactive({
  sender: '',
  receivers: '',
  cc: '',
  bcc: '',
  subject: ''
})
const toastMessage = ref('')
const toastType = ref('success')

const ruleForm = reactive({
  nickName: '',
  sender: '',
  receivers: [],
  cc: [],
  bcc: [],
  subject: '',
  domains: [],
  pickDomain: ""
})

const fileList = reactive([]);

const init = function () {
  if (Object.keys(globalStatus.userInfos) == 0 || globalStatus.userInfos === null || globalStatus.userInfos == undefined) {
    globalStatus.init(() => {
      ruleForm.sender = globalStatus.userInfos.account
      ruleForm.domains = globalStatus.userInfos.domains
      ruleForm.pickDomain = globalStatus.userInfos.domains[0]
      ruleForm.nickName = globalStatus.userInfos.name
    })
  } else {
    ruleForm.sender = globalStatus.userInfos.account
    ruleForm.domains = globalStatus.userInfos.domains
    ruleForm.pickDomain = globalStatus.userInfos.domains[0]
    ruleForm.nickName = globalStatus.userInfos.name
  }
}
init()

const checkEmail = function (str) {
  const re = /.+@.+\..+/;
  return re.test(str);
}

const showToast = (message, type = 'success') => {
  toastMessage.value = message
  toastType.value = type
  setTimeout(() => {
    toastMessage.value = ''
  }, 3000)
}

const addReceiver = () => {
  if (receiverInput.value.trim()) {
    ruleForm.receivers.push(receiverInput.value.trim())
    receiverInput.value = ''
  }
}

const removeReceiver = (index) => {
  ruleForm.receivers.splice(index, 1)
}

const addCC = () => {
  if (ccInput.value.trim()) {
    ruleForm.cc.push(ccInput.value.trim())
    ccInput.value = ''
  }
}

const removeCC = (index) => {
  ruleForm.cc.splice(index, 1)
}

const addBCC = () => {
  if (bccInput.value.trim()) {
    ruleForm.bcc.push(bccInput.value.trim())
    bccInput.value = ''
  }
}

const removeBCC = (index) => {
  ruleForm.bcc.splice(index, 1)
}

const validate = () => {
  errors.sender = ''
  errors.receivers = ''
  errors.cc = ''
  errors.bcc = ''
  errors.subject = ''

  if (typeof ruleForm.sender === "undefined" || ruleForm.sender === null || ruleForm.sender.trim() === "") {
    errors.sender = lang.err_sender_must
    return false
  }
  if (ruleForm.sender.includes("@")) {
    errors.sender = lang.only_prefix
    return false
  }

  for (let element of ruleForm.receivers) {
    if (!checkEmail(element)) {
      errors.receivers = lang.err_email_format
      return false
    }
  }

  for (let element of ruleForm.cc) {
    if (!checkEmail(element)) {
      errors.cc = lang.err_email_format
      return false
    }
  }

  for (let element of ruleForm.bcc) {
    if (!checkEmail(element)) {
      errors.bcc = lang.err_email_format
      return false
    }
  }

  if (!ruleForm.subject || ruleForm.subject.trim() === '') {
    errors.subject = lang.err_title_must
    return false
  }

  return true
}

const editorRef = shallowRef()

onBeforeUnmount(() => {
  const editor = editorRef.value
  if (editor == null) return
  editor.destroy()
})

const handleCreated = (editor) => {
  editorRef.value = editor
}

const send = function () {
  if (!validate()) {
    return
  }

  let objectTos = []
  for (let element of ruleForm.receivers) {
    objectTos.push({
      name: "",
      email: element
    })
  }

  let objectCcs = []
  for (let element of ruleForm.cc) {
    objectCcs.push({
      name: "",
      email: element
    })
  }

  let objectBccs = []
  for (let element of ruleForm.bcc) {
    objectBccs.push({
      name: "",
      email: element
    })
  }

  let text = editorRef.value.getText()

  http.post("/api/email/send", {
    from: {name: ruleForm.nickName, email: ruleForm.sender + "@" + ruleForm.pickDomain},
    to: objectTos,
    cc: objectCcs,
    bcc: objectBccs,
    subject: ruleForm.subject,
    text: text,
    html: valueHtml.value,
    attrs: fileList
  }).then(res => {
    if (res.errorNo === 0) {
      showToast(lang.succ_send, 'success')
      groupStore.name = lang.outbox
      groupStore.tag = '{"type":1,"status":-1}'
      router.replace({
        name: 'list',
      })
    } else {
      showToast(res.data, 'error')
    }
  })
}

const upload = function () {
  fileRef.value.dispatchEvent(new MouseEvent('click'))
}

const fileChange = function (e) {
  let files = e.target.files || e.dataTransfer.files;
  if (!files.length)
    return;
  for (let i = 0; i < files.length; i++) {
    const reader = new FileReader();
    reader.onload = function fileReadCompleted() {
      fileList.push({
        name: files[i].name,
        data: this.result
      })
    };
    reader.readAsDataURL(files[i]);
  }
}

const delFile = function (index) {
  fileList.splice(index, 1);
}
</script>

<style scoped>
</style>
