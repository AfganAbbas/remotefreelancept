<template>
  <button
    class="bg-neutral-200 rounded-full px-3 py-2 hover:bg-neutral-300"
    @click="store.setIncome(value)"
  >
    {{ asCurrency(value) }}
  </button>
</template>
<script lang="ts" setup>
import { asCurrency } from "@/utils.js";
import { useTaxesStore } from "@/stores";

const store = useTaxesStore();
defineProps({ value: { type: Number } });
</script>
