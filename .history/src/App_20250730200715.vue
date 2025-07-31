<script setup>
import { ref, computed, watch } from 'vue'

const types = {
  "3": {
    "name": "Router",
    "type": "ip_address",
    "array": true
  },
  "6": {
    "name": "DNS Servers",
    "type": "ip_address",
    "array": true
  },
  "15": {
    "name": "DNS Domain Name",
    "type": "string",
  },
  "42": {
    "name": "NTP Servers",
    "type": "ip_address",
    "array": true
  },
  "43": {
    "name": "Vendor-Specific Info",
    "type": "custom",
    "array": true,
    "fields": {
      "code": {
        "type": "u8",
        "hint": "Vendor-specific sub-option"
      },
      "payload": {
        "type": "string",
        "hint": "Enter payload",
      }
    }
  },
  "66": {
    "name": "TFTP Server",
    "type": "string",
  }
}

const typeEntries = Object.entries(types)

const selectedKey = ref("")
const inputValue = ref("")
const numberFormat = ref("dec")  // "dec" or "hex"

const selectedType = computed(() => types[selectedKey.value] || null)

watch(numberFormat, (newFormat, oldFormat) => {
  if (selectedType.value?.type !== 'number') return
  const num = parseInt(inputValue.value, oldFormat === 'hex' ? 16 : 10)
  if (!isNaN(num)) {
    inputValue.value = newFormat === 'hex' ? num.toString(16) : num.toString(10)
  }
})

</script>

<template>
  <h1>DHCP Builder</h1>
  <select v-model="selectedKey">
    <option disabled value="">Select an option</option>
    <option v-for="[key, value] in typeEntries" :key="key" :value="key">
      {{ key }} - {{ value.name }}
    </option>
  </select>

  <!-- <div v-if="selectedType" style="margin-top: 1em;">
    <input type="text" v-model="inputValue" placeholder="Enter value" />
    <span v-if="selectedType.array" style="margin-left: 0.5em; color: gray;">
      One or more values (comma-separated)
    </span>
  </div> -->

</template>

<style scoped></style>
