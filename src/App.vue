<script setup>
import { ref, computed, watch } from 'vue'

/*
There should be something to allow override of checks. For example, in Calix, we restricted option 66 strictly to
what the RFC outlined. But that bit us in the butt later when people had put a generic string in. So for every option
there should be the ability to discard all automatic behavior.
*/

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
  },
  "120": {
    "name": "SIP Servers",
    "type": "string",
    "hint": "Enter a comma-separated list of FQDNs or IP addresses. Examples: '192.0.2.47,198.18.6.2' or 'sip.example.com,slurp.example.info'",
    "array": true
  },
  "128": {
    "name": "Polycom Phone VLAN ID",
    "type": "string",
    "default": "VLAN-A=",
    "help": "Typical format is 'VLAN-A=xxxx;' where xxxx is VLAN number (decimal). Examples: 'VLAN-A=2;' or 'VLAN-A=4088;'. Polycom documentation refers to this as a DVD (DHCP VLAN Discovery) string",
    "help_url": "https://docs.poly.com/bundle/ccx-opensip-pg-current/page/c-ucs-ag-valid-dvd-string-dhcp-options.html"
  },
  "132": {
    "name": "VoIP VLAN ID",
    "type": "number",
    "help": "VLAN ID (in decimal format)",
    "min": 1,
    "max": 4094
  },
  "144": {
    "name": "Polycom Phone VLAN ID",
    "type": "string",
    "default": "VLAN-A=",
    "help": "Typical format is 'VLAN-A=xxxx;' where xxxx is VLAN number (decimal). Examples: 'VLAN-A=2;' or 'VLAN-A=4088;'. Polycom documentation refers to this as a DVD (DHCP VLAN Discovery) string",
    "help_url": "https://docs.poly.com/bundle/ccx-opensip-pg-current/page/c-ucs-ag-valid-dvd-string-dhcp-options.html"
  },
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
    inputValue.value = newFormat === 'hex' ? num.toString(16).padStart(2, '0') : num.toString(10)
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

// Dynamic validation ---
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
      errorMessage.value = "Enter a valid hex value between 00 and FF"
      return
    }
    const num = parseInt(val, 16)
    if (num < 0 || num > 255) {
      errorMessage.value = "Hex value must be between 00 and FF"
      return
    }
  }

  if (numberFormat.value === 'dec') {
    const num = Number(val)
    if (isNaN(num) || num < 0 || num > 255) {
      errorMessage.value = "Decimal number must be between 0 and 255";
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
