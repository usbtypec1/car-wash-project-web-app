<template>
  <ClientOnly>
    <Toast/>
    <ConfirmDialog class="mx-3"/>
    <NuxtLayout>
      <NuxtPage/>
    </NuxtLayout>
  </ClientOnly>
</template>

<style>
.page-enter-active,
.page-leave-active {
  transition: all 0.15s;
}

.page-enter-from,
.page-leave-to {
  opacity: 0;
  filter: blur(1rem);
}
</style>

<script setup>
import { useWebApp } from 'vue-tg'
import { userIdKey } from '~/utils/keys'

const webApp = useWebApp()

const userId = webApp?.initDataUnsafe?.user?.id

provide(userIdKey, userId)
</script>
