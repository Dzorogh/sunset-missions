<script setup lang="ts">
import { darkTheme, NLayout } from 'naive-ui';
</script>

<template>
  <n-config-provider :theme="darkTheme">
    <n-global-style />
    <n-layout position="absolute">

      <n-layout style="padding: 24px;">
        <SelectRandomItem msg="Vite + Vue" />
      </n-layout>
    </n-layout>

  </n-config-provider>
</template>

<style scoped></style>
