<template>
  <div class="w-full h-full bg-gray-100 flex flex-col">
    <!-- Steps Indicator -->
    <div class="bg-white shadow-sm p-6">
      <div class="flex items-center justify-center gap-4 max-w-5xl mx-auto">
        <div v-for="(step, index) in steps" :key="index" class="flex items-center">
          <div class="flex flex-col items-center">
            <div :class="[
              'w-10 h-10 rounded-full flex items-center justify-center font-bold transition',
              active > index ? 'bg-green-500 text-white' : active === index ? 'bg-blue-500 text-white' : 'bg-gray-300 text-gray-600'
            ]">
              <span v-if="active > index" class="iconify" data-icon="mdi:check"></span>
              <span v-else>{{ index + 1 }}</span>
            </div>
            <div class="mt-2 text-sm text-center whitespace-nowrap">{{ step }}</div>
          </div>
          <div v-if="index < steps.length - 1" :class="[
            'w-16 h-1 mx-2',
            active > index ? 'bg-green-500' : 'bg-gray-300'
          ]"></div>
        </div>
      </div>
    </div>

    <!-- Content Area -->
    <div class="flex-1 overflow-y-auto p-8">
      <!-- Step 0: Welcome -->
      <div v-if="active === 0" class="max-w-2xl mx-auto bg-white rounded-lg shadow p-8">
        <h2 class="text-3xl font-bold mb-4">{{ lang.tks_pmail }}</h2>
        <p class="text-gray-600">{{ lang.guid_desc }}</p>
      </div>

      <!-- Step 1: Database -->
      <div v-if="active === 1" class="max-w-2xl mx-auto bg-white rounded-lg shadow p-8">
        <h2 class="text-2xl font-bold mb-2">{{ lang.select_db }}</h2>
        <p class="text-gray-600 mb-6">{{ lang.db_desc }}</p>
        
        <div class="space-y-4">
          <div>
            <label class="block text-sm font-bold mb-2">{{ lang.type }}</label>
            <select 
              v-model="dbSettings.type" 
              @change="dbSettings.dsn = ''"
              class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
            >
              <option value="mysql">MySQL</option>
              <option value="sqlite">SQLite3</option>
              <option value="postgres">PostgreSQL</option>
            </select>
          </div>

          <div v-if="dbSettings.type === 'mysql'">
            <label class="block text-sm font-bold mb-2">{{ lang.mysql_dsn }}</label>
            <textarea 
              v-model="dbSettings.dsn"
              rows="2"
              placeholder="root:12345@tcp(127.0.0.1:3306)/pmail?parseTime=True&loc=Local"
              class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
            ></textarea>
          </div>

          <div v-if="dbSettings.type === 'postgres'">
            <label class="block text-sm font-bold mb-2">{{ lang.pg_dsn }}</label>
            <textarea 
              v-model="dbSettings.dsn"
              rows="2"
              placeholder="postgres://postgres:12345@127.0.0.1:5432/pmail?sslmode=disable"
              class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
            ></textarea>
          </div>

          <div v-if="dbSettings.type === 'sqlite'">
            <label class="block text-sm font-bold mb-2">{{ lang.sqlite_db_path }}</label>
            <input 
              v-model="dbSettings.dsn"
              placeholder="./config/pmail.db"
              class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>
        </div>
      </div>

      <!-- Step 2: Admin Password -->
      <div v-if="active === 2" class="max-w-2xl mx-auto bg-white rounded-lg shadow p-8">
        <h2 class="text-2xl font-bold mb-6">{{ lang.setAdminPassword }}</h2>
        
        <div class="space-y-4">
          <div>
            <label class="block text-sm font-bold mb-2">{{ lang.admin_account }}</label>
            <input 
              v-model="adminSettings.account"
              :disabled="adminSettings.hadSeted"
              placeholder="admin"
              class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:bg-gray-100"
            />
          </div>

          <div>
            <label class="block text-sm font-bold mb-2">{{ lang.password }}</label>
            <input 
              type="password"
              v-model="adminSettings.password"
              :disabled="adminSettings.hadSeted"
              class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:bg-gray-100"
            />
          </div>

          <div>
            <label class="block text-sm font-bold mb-2">{{ lang.enter_again }}</label>
            <input 
              type="password"
              v-model="adminSettings.password2"
              :disabled="adminSettings.hadSeted"
              class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:bg-gray-100"
            />
          </div>
        </div>
      </div>

      <!-- Step 3: Domain -->
      <div v-if="active === 3" class="max-w-2xl mx-auto bg-white rounded-lg shadow p-8">
        <h2 class="text-2xl font-bold mb-6">{{ lang.SetDomail }}</h2>
        
        <div class="space-y-4">
          <div>
            <label class="block text-sm font-bold mb-2">{{ lang.smtp_domain }}</label>
            <div class="flex items-center">
              <span class="bg-gray-200 px-3 py-2 border border-r-0 rounded-l">smtp.</span>
              <input 
                v-model="domainSettings.smtp_domain"
                placeholder="domain.com"
                class="flex-1 px-3 py-2 border rounded-r focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </div>
          </div>

          <div>
            <label class="block text-sm font-bold mb-2">{{ lang.web_domain }}</label>
            <input 
              v-model="domainSettings.web_domain"
              placeholder="pmail.domain.com"
              class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
            />
          </div>

          <div>
            <div class="flex items-center justify-between mb-2">
              <label class="block text-sm font-bold">{{ lang.multi_domain_setting }}</label>
              <button 
                @click="addDomain"
                class="bg-green-500 hover:bg-green-600 text-white w-8 h-8 rounded-full transition"
              >
                +
              </button>
            </div>
            <p class="text-sm text-gray-600 mb-2">{{ lang.multi_domain_setting_desc }}</p>
            <div class="space-y-2">
              <input 
                v-for="(item, i) in domainSettings.multi_domain" 
                :key="i"
                v-model="domainSettings.multi_domain[i]"
                :placeholder="'domain' + i + '.com'"
                class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </div>
          </div>
        </div>
      </div>

      <!-- Step 4: DNS -->
      <div v-if="active === 4" class="max-w-5xl mx-auto">
        <div class="bg-white rounded-lg shadow p-8 mb-6">
          <h2 class="text-2xl font-bold mb-2">{{ lang.setDNS }}</h2>
          <p class="text-gray-600 mb-6">{{ lang.dns_desc }}</p>
        </div>
        
        <div v-for="(info, domain) in dnsInfos" :key="domain" class="bg-white rounded-lg shadow p-6 mb-4">
          <h3 class="text-xl font-bold mb-4">{{ domain }}</h3>
          <div class="overflow-x-auto">
            <table class="w-full border-collapse border">
              <thead class="bg-gray-50">
                <tr>
                  <th class="border p-2 text-left w-28">HOSTNAME</th>
                  <th class="border p-2 text-left w-28">TYPE</th>
                  <th class="border p-2 text-left">VALUE</th>
                  <th class="border p-2 text-left w-28">TTL</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="row in info" :key="row.host + row.type">
                  <td class="border p-2">
                    <span :title="lang.dns_root_desc" v-if="row.host === '' || row.host === '@'">{{ row.host }}</span>
                    <span v-else>{{ row.host }}</span>
                  </td>
                  <td class="border p-2">{{ row.type }}</td>
                  <td class="border p-2">
                    <span :title="row.tips" v-if="row.tips !== ''">{{ row.value }}</span>
                    <span v-else>{{ row.value }}</span>
                  </td>
                  <td class="border p-2">{{ row.ttl }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- Step 5: SSL -->
      <div v-if="active === 5" class="max-w-3xl mx-auto">
        <div v-if="sslSettings.type === '0' && port !== 80" class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded mb-6">
          <div class="font-bold">Warning!</div>
          <div>{{ lang.autoSSLWarn }}</div>
        </div>

        <div class="bg-white rounded-lg shadow p-8">
          <h2 class="text-2xl font-bold mb-2">{{ lang.setSSL }}</h2>
          <p class="text-gray-600 mb-6">{{ lang.setSSL }}</p>
          
          <div class="space-y-4">
            <div>
              <label class="block text-sm font-bold mb-2">{{ lang.type }}</label>
              <select 
                v-model="sslSettings.type"
                :disabled="dnsChecking"
                class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:bg-gray-100"
              >
                <option value="0">{{ lang.ssl_auto }}</option>
                <option value="1">{{ lang.ssl_manuallyf }}</option>
              </select>
            </div>

            <div v-if="sslSettings.type === '0'">
              <label class="block text-sm font-bold mb-2">{{ lang.ssl_challenge_type }}</label>
              <div class="flex items-center gap-2">
                <select 
                  v-model="sslSettings.challenge"
                  :disabled="dnsChecking"
                  class="flex-1 px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500 disabled:bg-gray-100"
                >
                  <option value="http">{{ lang.ssl_auto_http }}</option>
                  <option value="dns">{{ lang.ssl_auto_dns }}</option>
                </select>
                <span :title="lang.challenge_typ_desc" class="text-xl font-bold cursor-help">?</span>
              </div>
            </div>

            <div v-if="sslSettings.type === '1'">
              <label class="block text-sm font-bold mb-2">{{ lang.ssl_key_path }}</label>
              <input 
                v-model="sslSettings.key_path"
                placeholder="./config/ssl/private.key"
                class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </div>

            <div v-if="sslSettings.type === '1'">
              <label class="block text-sm font-bold mb-2">{{ lang.ssl_crt_path }}</label>
              <input 
                v-model="sslSettings.crt_path"
                placeholder="./config/ssl/public.crt"
                class="w-full px-3 py-2 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
            </div>
          </div>
        </div>

        <div v-if="dnsChecking" class="bg-white rounded-lg shadow p-8 mt-4">
          <label class="block text-lg font-bold mb-4">{{ lang.dns_desc }}</label>
          <div v-if="sslSettings.paramsList.length === 0" class="text-center py-8 text-gray-500">
            Loading...
          </div>
          <div v-else class="overflow-x-auto">
            <table class="w-full border-collapse border">
              <thead class="bg-gray-50">
                <tr>
                  <th class="border p-2 text-left w-28">HOSTNAME</th>
                  <th class="border p-2 text-left w-28">TYPE</th>
                  <th class="border p-2 text-left">VALUE</th>
                  <th class="border p-2 text-left w-28">TTL</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="row in sslSettings.paramsList" :key="row.host">
                  <td class="border p-2">{{ row.host }}</td>
                  <td class="border p-2">{{ row.type }}</td>
                  <td class="border p-2">
                    <span :title="row.tips" v-if="row.tips !== ''">{{ row.value }}</span>
                    <span v-else>{{ row.value }}</span>
                  </td>
                  <td class="border p-2">{{ row.ttl }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <!-- Navigation Button -->
    <div class="bg-white border-t p-4 flex justify-center">
      <button 
        @click="next"
        :disabled="fullscreenLoading"
        class="bg-blue-500 hover:bg-blue-600 text-white px-8 py-3 rounded font-bold transition disabled:opacity-50 disabled:cursor-not-allowed"
      >
        {{ lang.next }}
      </button>
    </div>

    <!-- Loading Overlay -->
    <div v-if="fullscreenLoading" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div class="bg-white rounded-lg p-8 text-center">
        <div class="animate-spin rounded-full h-16 w-16 border-b-2 border-blue-500 mx-auto mb-4"></div>
        <div class="text-lg font-bold">{{ waitDesc }}</div>
      </div>
    </div>

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
import {reactive, ref} from 'vue'
import lang from '../i18n/i18n';
import axios from 'axios'
import {http} from "@/utils/axios";

const waitDesc = ref(lang.wait_desc);
const toastMessage = ref('')
const toastType = ref('success')

const showToast = (message, type = 'success') => {
  toastMessage.value = message
  toastType.value = type
  setTimeout(() => {
    toastMessage.value = ''
  }, 3000)
}

const steps = [
  lang.welcome,
  lang.setDatabase,
  lang.setAdminPassword,
  lang.SetDomail,
  lang.setDNS,
  lang.setSSL
]

const adminSettings = reactive({
  "account": "admin",
  "password": "",
  "password2": "",
  "hadSeted": false
})

const dbSettings = reactive({
  "type": "sqlite",
  "dsn": "./config/pmail.db",
  "lable": ""
})

const domainSettings = reactive({
  "web_domain": "",
  "smtp_domain": "",
  "multi_domain": []
})

const sslSettings = reactive({
  "type": "0",
  "challenge": "http",
  "key_path": "./config/ssl/private.key",
  "crt_path": "./config/ssl/public.crt",
  "paramsList": [],
})

const active = ref(0)
const fullscreenLoading = ref(false)
const dnsChecking = ref(false)
const dnsInfos = ref({})
const port = ref(80)

const addDomain = () => {
  domainSettings.multi_domain.push([])
}

const setPassword = () => {
  if (adminSettings.hadSeted) {
    active.value++;
    getDomainConfig();
    return;
  }

  if (adminSettings.password !== adminSettings.password2) {
    showToast(lang.err_pwd_diff, 'error')
  } else {
    http.post("/api/setup", {
      "action": "set",
      "step": "password",
      "account": adminSettings.account,
      "password": adminSettings.password
    }).then((res) => {
      if (res.errorNo !== 0) {
        showToast(res.errorMsg, 'error')
      } else {
        active.value++;
        getDomainConfig();
      }
    })
  }
}

const getPassword = () => {
  http.post("/api/setup", {"action": "get", "step": "password"}).then((res) => {
    if (res.errorNo !== 0) {
      showToast(res.errorMsg, 'error')
    } else {
      adminSettings.hadSeted = res.data !== ""
      if (adminSettings.hadSeted) {
        adminSettings.account = res.data
        adminSettings.password = "*******"
        adminSettings.password2 = "*******"
      }
    }
  })
}

const getDbConfig = () => {
  http.post("/api/setup", {"action": "get", "step": "database"}).then((res) => {
    if (res.errorNo !== 0) {
      showToast(res.errorMsg, 'error')
    } else {
      dbSettings.type = res.data.db_type;
      dbSettings.dsn = res.data.db_dsn;
    }
  })
}

const getDomainConfig = () => {
  http.post("/api/setup", {"action": "get", "step": "domain"}).then((res) => {
    if (res.errorNo !== 0) {
      showToast(res.errorMsg, 'error')
    } else {
      domainSettings.web_domain = res.data.web_domain;
      domainSettings.smtp_domain = res.data.smtp_domain;
      domainSettings.multi_domain = res.data.domains;
    }
  })
}

const setDbConfig = () => {
  if (dbSettings.type === "sqlite" && !dbSettings.dsn) dbSettings.dsn = "./config/pmail.db";
  else if (!dbSettings.dsn) {
    showToast(lang.err_db_dsn_empty, 'error')
    return
  }
  http.post("/api/setup", {
    "action": "set",
    "step": "database",
    "db_type": dbSettings.type,
    "db_dsn": dbSettings.dsn
  }).then((res) => {
    if (res.errorNo !== 0) {
      showToast(res.errorMsg, 'error')
    } else {
      active.value++;
      getPassword();
    }
  })
}

const getDNSConfig = () => {
  http.post("/api/setup", {"action": "get", "step": "dns"}).then((res) => {
    if (res.errorNo !== 0) {
      showToast(res.errorMsg, 'error')
    } else {
      dnsInfos.value = res.data
    }
  })
}

const getSSLConfig = () => {
  http.post("/api/setup", {"action": "get", "step": "ssl"}).then((res) => {
    if (res.errorNo !== 0) {
      showToast(res.errorMsg, 'error')
    } else {
      sslSettings.type = res.data.type
      if (sslSettings.type === "2") {
        sslSettings.type = "0"
        sslSettings.challenge = "dns"
      }
      port.value = res.data.port
    }
  })
}

const setSSLConfig = () => {
  fullscreenLoading.value = true;

  let sslType = sslSettings.type;
  if (sslType === "0" && sslSettings.challenge === "dns") {
    sslType = "2"
  }

  http.post("/api/setup", {
    "action": "set",
    "step": "ssl",
    "ssl_type": sslType,
    "key_path": sslSettings.key_path,
    "crt_path": sslSettings.crt_path
  }).then((res) => {
    if (res.errorNo !== 0) {
      fullscreenLoading.value = false;
      showToast(res.errorMsg, 'error')
    } else {
      if (sslType == 2) {
        fullscreenLoading.value = false;
        dnsChecking.value = true;
        getSSLDNSParams();
      }
      checkStatus();
    }
  })
}

const checkStatus = () => {
  axios.post("/api/ping", {}).then((res) => {
    if (res.data.errorNo !== 0) {
      setTimeout(function () {
        checkStatus()
      }, 1000);
    } else {
      if (sslSettings.type === '1') {
        window.location.href = "http://" + domainSettings.web_domain;
      } else {
        window.location.href = "https://" + domainSettings.web_domain;
      }
    }
  }).catch(() => {
    setTimeout(function () {
      checkStatus()
    }, 1000);
  })
}

const setDomainConfig = () => {
  http.post("/api/setup", {
    "action": "set",
    "step": "domain",
    "web_domain": domainSettings.web_domain,
    "smtp_domain": domainSettings.smtp_domain,
    "multi_domain": domainSettings.multi_domain.join(",")
  }).then((res) => {
    if (res.errorNo !== 0) {
      showToast(res.errorMsg, 'error')
    } else {
      active.value++;
      getDNSConfig();
    }
  })
}

const getSSLDNSParams = () => {
  http.post("/api/setup", {"action": "getParams", "step": "ssl"}).then((res) => {
    if (res.errorNo !== 0) {
      showToast(res.errorMsg, 'error')
    } else {
      sslSettings.paramsList = res.data
    }
  })

  if (sslSettings.paramsList.length === 0) {
    setTimeout(function () {
      getSSLDNSParams()
    }, 1000);
  }
}

const next = () => {
  switch (active.value) {
    case 0:
      active.value++
      getDbConfig();
      break
    case 1:
      setDbConfig();
      break;
    case 2:
      setPassword();
      break;
    case 3:
      setDomainConfig();
      break;
    case 4:
      getSSLConfig();
      active.value++
      break
    case 5:
      if (dnsChecking.value) {
        fullscreenLoading.value = true;
        waitDesc.value = lang.dns_challenge_wait;
      } else {
        setSSLConfig();
      }
      break
  }
}
</script>

<style scoped>
</style>
