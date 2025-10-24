<template>
  <div>
    <div v-if="Object.keys(pluginList).length === 0" class="text-gray-500 text-center py-8">
      No plugins available
    </div>
    
    <div v-else>
      <!-- Tab Headers -->
      <div class="flex border-b border-gray-300 mb-4">
        <button 
          v-for="(src, name) in pluginList" 
          :key="src"
          @click="activePlugin = name"
          :class="[
            'px-4 py-2 border-b-2 transition',
            activePlugin === name ? 'border-blue-500 text-blue-600 font-semibold' : 'border-transparent text-gray-600 hover:text-gray-800'
          ]"
        >
          {{ name }}
        </button>
      </div>

      <!-- Tab Content -->
      <div v-for="(src, name) in pluginList" :key="src" v-show="activePlugin === name">
        <iframe :src="src" class="w-full border-0" style="min-height: 500px;"></iframe>
      </div>
    </div>
  </div>
</template>

<script setup>
import {reactive, ref} from 'vue'
import {http} from "@/utils/axios";

const pluginList = reactive({})
const activePlugin = ref('')

http.get('/api/plugin/list').then(res => {
  if (res.data != null && res.data.length > 0) {
    for (let i = 0; i < res.data.length; i++) {
      let name = res.data[i];
      pluginList[name] = "/api/plugin/settings/" + name + "/index.html";
    }
    // Set first plugin as active
    activePlugin.value = res.data[0]
  }
})
</script>

<style scoped>
</style>
