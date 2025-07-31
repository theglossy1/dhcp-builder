<script setup>
const types = {
  "3": {
    "name": "router",
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
    "fields": [
      "code", "hex"
    ]
  },
}
</script>

<template>
  <h1>DHCP Builder</h1>
  <p>
    Visit <a href="https://vuejs.org/" target="_blank" rel="noopener">vuejs.org</a> to read the
    documentation
  </p>
</template>

<style scoped></style>
