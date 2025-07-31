<script setup>
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
      "code": "number",
      "payload": "hex/ascii"
    }
  },
  "66": {
    "name": "TFTP Server",
    "type": "string"
  }
}

// Convert the object to an array of entries for easier looping
const typeEntries = Object.entries(types)
</script>

<template>
  <h1>DHCP Builder</h1>
  <p>
    <select>
      <option disabled value="">Select an option</option>
      <option v-for="[key, value] in typeEntries" :key="key" :value="key">
        {{ key }} - {{ value.name }}
      </option>
    </select>
  </p>
</template>

<style scoped></style>
