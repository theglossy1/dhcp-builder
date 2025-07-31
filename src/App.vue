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
    "type": "number",
  }
}

const typeEntries = Object.entries(types)

const selectedKey = ref("")
const inputValue = ref("")
const numberFormat = ref("dec")
const stringFormat = ref("text")
const errorMessage = ref("")

const selectedType = computed(() => types[selectedKey.value] || null)

// --- Watchers for format switching (existing logic) ---

watch(numberFormat, (newFormat, oldFormat) => {
  if (selectedType.value?.type !== 'number') return
  const num = parseInt(inputValue.value, oldFormat === 'hex' ? 16 : 10)
  if (!isNaN(num)) {
    inputValue.value = newFormat === 'hex' ? num.toString(16) : num.toString(10)
  }
})

watch(stringFormat, (newFormat, oldFormat) => {
  if (selectedType.value?.type !== 'string') return
  if (newFormat === oldFormat) return

  if (newFormat === 'hex') {
    inputValue.value = Array.from(inputValue.value)
      .map(char => char.charCodeAt(0).toString(16).padStart(2, '0'))
      .join('')
  } else if (newFormat === 'text') {
    const hex = inputValue.value
    if (/^[0-9a-f]*$/i.test(hex) && hex.length % 2 === 0) {
      let text = ''
      for (let i = 0; i < hex.length; i += 2) {
        const code = parseInt(hex.slice(i, i + 2), 16)
        text += String.fromCharCode(code)
      }
      inputValue.value = text
    }
  }
})

// --- NEW: Dynamic validation ---
watch([inputValue, numberFormat, selectedType], () => {
  if (selectedType.value?.type !== 'number') {
    errorMessage.value = ""
    return
  }

  const val = inputValue.value.trim()

  // Don't validate if the input is empty
  if (val === "") {
    errorMessage.value = ""
    return
  }

  if (numberFormat.value === 'hex') {
    if (!/^[0-9a-f]{1,2}$/i.test(val)) {
      errorMessage.value = "Enter a valid hex value (00-ff)"
      return
    }
    const num = parseInt(val, 16)
    if (num < 0 || num > 255) {
      errorMessage.value = "Hex value must be between 00 and ff"
      return
    }
  }

  if (numberFormat.value === 'dec') {
    const num = Number(val)
    if (!/^\d{1,3}$/.test(val) || isNaN(num) || num < 0 || num > 255) {
      errorMessage.value = "Enter a valid decimal number (0-255)";
      return
    }
  }

  errorMessage.value = ""
})

// --- Submit handler ---
function verify() {
  const data = {
    optionCode: selectedKey.value,
    name: selectedType.value?.name || '',
    type: selectedType.value?.type || '',
    value: inputValue.value,
    format: selectedType.value?.type === 'number'
      ? numberFormat.value
      : selectedType.value?.type === 'string'
        ? stringFormat.value
        : undefined
  }

  console.log('Submitted:', data)
}

</script>

<template>
  <h1>DHCP Builder</h1>

  <select v-model="selectedKey">
    <option disabled value="">Select an option</option>
    <option v-for="[key, value] in typeEntries" :key="key" :value="key">
      {{ key }} - {{ value.name }}
    </option>
  </select>

  <div v-if="selectedType" style="margin-top: 1em;">
    <input type="text" v-model="inputValue" placeholder="Enter value" />
    <!-- <input :type="selectedType.type === 'number' ? 'number' : 'text'" v-model="inputValue" placeholder="Enter value"
      :min="selectedType.type === 'number' ? 0 : null" :max="selectedType.type === 'number' ? 255 : null" /> -->
    <span v-if="selectedType.array" style="margin-left: 0.5em; color: gray;">
      comma-separate multiple values
    </span>
    <div v-if="errorMessage" style="color: red; margin-top: 0.5em;">
      {{ errorMessage }}
    </div>


    <div v-if="selectedType.type === 'number'" style="margin-top: 0.5em;">
      <label><input type="radio" value="dec" v-model="numberFormat" /> dec</label>
      <label style="margin-left: 1em;"><input type="radio" value="hex" v-model="numberFormat" /> hex</label>
    </div>

    <div v-if="selectedType.type === 'string'" style="margin-top: 0.5em;">
      <label><input type="radio" value="text" v-model="stringFormat" /> text</label>
      <label style="margin-left: 1em;"><input type="radio" value="hex" v-model="stringFormat" /> hex</label>
    </div>

    <div style="margin-top: 1em;">
      <button id="subnet" @click="verify">Submit</button>
    </div>
  </div>
</template>
<style scoped></style>
