<script setup lang="ts">
import AppSidebar from '@/components/AppSidebar.vue'
import Spacer from '@/components/Spacer.vue'
import { SidebarProvider, SidebarTrigger } from '@/components/ui/sidebar'
import { RouterView } from 'vue-router'
</script>

<template>
  <SidebarProvider>
    <AppSidebar />

    <SidebarTrigger />
    <main class="w-full min-h-screen p-4">

      <Spacer size="100" />
      <div class="grid grid-cols-7 grid-rows-5 gap-4">
        <!-- Colonna Sinistra / Filtri (Fissa) -->
        <div class="row-span-5">1</div>

        <div class="col-span-5 row-span-5 flex flex-col items-center justify-center min-h-full">
          <RouterView />
        </div>

        <div class="row-span-5 col-start-7">3</div>

      </div>
    </main>
  </SidebarProvider>
</template>