<script lang="ts" setup>
import { HidDevice } from "@redfernelec/tauri-plugin-hid-api";
import { ref } from "vue";

const props = defineProps<{
  device: HidDevice;
}>();

defineEmits<{
  (e: "disconnect", device: HidDevice): void;
}>();

const write_string = ref("");
const read_string = ref("");

async function write() {
  const device = props.device;
  
  // Split the string into an array of numbers
  const numbers = write_string.value.split(",").map(Number);
  // Create a Uint8Array from the numbers
  const uint8Array = new Uint8Array(numbers);
  // Convert the Uint8Array to an ArrayBuffer
  const arrayBuffer = uint8Array.buffer;
  try {
    await device.write(arrayBuffer);
  } catch (e) {
    console.error("Error writing to device", e);
    read_string.value = "Error writing to device";
  }
  write_string.value = "";
}

async function read() {
  const device = props.device;
  try {
    let data = new Uint8Array(await device.read(100));
    if (data.length === 0) {
      read_string.value = "No data available";
      return;
    }
    read_string.value = data.join(", ");
  } catch (e) {
    console.error("Error reading from device", e);
    read_string.value = "Error reading from device";
  }
}

</script>

<template>
  <div class="device-connected">
    <h1>{{ device.productString }} ({{ device.manufacturerString }})</h1>
    <p>Vendor ID: 0x{{ device.vendorId.toString(16).toUpperCase().padStart(4, '0') }}</p>
    <p>Product ID: 0x{{ device.productId.toString(16).toUpperCase().padStart(4, '0') }}</p>
    <hr>
    <input type="text" placeholder="E.g. 0,2,4,128,55" v-model="write_string" /><button @click="write">Write</button>
    <hr>
    <div><button @click="read">Read</button>
    <p> {{read_string}} </p></div>
    <hr>
    <button @click="$emit('disconnect', device)">Disconnect</button>
  </div>
</template>

<style scoped>
  .device-connected {
    border: 1px solid #ccc;
    padding: 10px;
    margin: 10px;
    border-radius: 5px;
    border-color: #ccc;
    button {
      margin-left: 10px;
    }
    hr {
      background-color: #ccc;
      border-width: 0;
      height: 1px;
    }
  }

  .device-connected:hover {
    border-color: #396cd8;
  }
</style>