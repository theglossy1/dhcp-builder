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
</script>

<template>
  <h1>DHCP Builder</h1>
  <p>
    <select>
      <option value=""></option>
    </select>
  </p>
</template>

<style scoped></style>
