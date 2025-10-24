<template>
  <div>
    <div v-for="(item, index) in data" :key="item.id" class="mb-2">
      <!-- Root/Parent Item -->
      <div class="flex items-center gap-2 mb-2 p-2 bg-gray-50 rounded">
        <span v-if="item.id !== -1" class="flex-1 font-semibold">{{ item.label }}</span>
        <input 
          v-if="item.id === -1"
          v-model="item.label"
          @blur="onInputBlur(item, index)"
          @keyup.enter="onInputBlur(item, index)"
          class="flex-1 px-2 py-1 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="Group name"
          autofocus
        />
        <button
          v-if="item.id !== 0"
          @click="add(item)"
          class="bg-blue-500 hover:bg-blue-600 text-white w-8 h-8 rounded-full transition"
          title="Add child"
        >
          +
        </button>
        <button
          v-if="item.id !== 0"
          @click="del(item, index)"
          class="bg-red-500 hover:bg-red-600 text-white w-8 h-8 rounded-full transition"
          title="Delete"
        >
          -
        </button>
      </div>

      <!-- Children Items -->
      <div v-if="item.children && item.children.length > 0" class="ml-6">
        <div v-for="(child, childIndex) in item.children" :key="child.id" class="flex items-center gap-2 mb-2 p-2 bg-gray-100 rounded">
          <span v-if="child.id !== -1" class="flex-1">{{ child.label }}</span>
          <input 
            v-if="child.id === -1"
            v-model="child.label"
            @blur="onInputBlur(child, childIndex, item)"
            @keyup.enter="onInputBlur(child, childIndex, item)"
            class="flex-1 px-2 py-1 border rounded focus:outline-none focus:ring-2 focus:ring-blue-500"
            placeholder="Sub-group name"
            autofocus
          />
          <button
            @click="delChild(item, childIndex)"
            class="bg-red-500 hover:bg-red-600 text-white w-8 h-8 rounded-full transition"
            title="Delete"
          >
            -
          </button>
        </div>
      </div>
    </div>

    <button 
      @click="addRoot"
      class="mt-4 bg-blue-500 hover:bg-blue-600 text-white px-6 py-2 rounded transition"
    >
      <span class="iconify mr-2" data-icon="mdi:plus"></span>{{ lang.add_group }}
    </button>

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
import { reactive, ref } from "vue";
import lang from "../i18n/i18n";
import { http } from "@/utils/axios";

const data = reactive([]);
const toastMessage = ref('')
const toastType = ref('success')

const showToast = (message, type = 'success') => {
  toastMessage.value = message
  toastType.value = type
  setTimeout(() => {
    toastMessage.value = ''
  }, 3000)
}

http.get("/api/group").then((res) => {
  data.push(...res.data);
});

const refreshData = () => {
  http.get("/api/group").then((res) => {
    data.splice(0, data.length);
    data.push(...res.data);
  });
}

const del = function (item, index) {
  if (item.id !== -1) {
    http.post("/api/group/del", { id: item.id }).then((res) => {
      if (res.errorNo !== 0) {
        showToast(res.errorMsg, 'error')
      } else {
        data.splice(index, 1);
        showToast('Deleted successfully', 'success')
      }
    });
  } else {
    data.splice(index, 1);
  }
};

const delChild = function (parent, childIndex) {
  const child = parent.children[childIndex];
  if (child.id !== -1) {
    http.post("/api/group/del", { id: child.id }).then((res) => {
      if (res.errorNo !== 0) {
        showToast(res.errorMsg, 'error')
      } else {
        parent.children.splice(childIndex, 1);
        showToast('Deleted successfully', 'success')
      }
    });
  } else {
    parent.children.splice(childIndex, 1);
  }
};

const add = function (item) {
  if (item.children == null) {
    item.children = [];
  }
  item.children.push({
    children: [],
    label: "",
    id: -1,
    parent_id: item.id,
  });
};

const addRoot = function () {
  data.push({
    children: [],
    label: "",
    id: -1,
    parent_id: 0,
  });
};

const onInputBlur = function (item, index, parent = null) {
  if (item.label !== "") {
    http.post("/api/group/add", { name: item.label, parent_id: item.parent_id })
      .then((res) => {
        if (res.errorNo !== 0) {
          showToast(res.errorMsg, 'error')
        } else {
          refreshData();
          showToast('Added successfully', 'success')
        }
      });
  } else {
    // Remove empty item
    if (parent) {
      parent.children.splice(index, 1);
    } else {
      data.splice(index, 1);
    }
  }
};
</script>

<style scoped>
</style>
