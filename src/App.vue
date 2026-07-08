<script setup lang="ts">
import { ref } from 'vue'
import GatePage from './Gatepage.vue'
import BirthdaySite from './Birthdaysite.vue'

const entered = ref(false)

function onEnter() {
  entered.value = true
}
</script>

<template>
  <GatePage v-if="!entered" @enter="onEnter" />
  <BirthdaySite v-else />
</template>
