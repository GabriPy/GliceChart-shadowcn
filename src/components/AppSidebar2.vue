




<script setup>
import { LayoutDashboard, Menu } from '@lucide/vue'

const items = [
    {
        title: 'Dashboard',
        to: '/',
        icon: LayoutDashboard,
    },
    {
        title: 'Impostazioni',
        to: '/settings',
        icon: LayoutDashboard,
    },

]
</script>

<template>
    <button data-drawer-target="separator-sidebar" data-drawer-toggle="separator-sidebar"
        aria-controls="separator-sidebar" type="button"
        class="text-heading bg-transparent box-border border border-transparent hover:bg-neutral-secondary-medium focus:ring-4 focus:ring-neutral-tertiary font-medium leading-5 rounded-base ms-3 mt-3 text-sm p-2 focus:outline-none inline-flex sm:hidden">
        <span class="sr-only">Open sidebar</span>
        <Menu class="size-6" aria-hidden="true" />
    </button>

    <aside id="separator-sidebar"
        class="fixed top-0 left-0 z-40 w-64 h-screen transition-transform -translate-x-full sm:translate-x-0"
        aria-label="Sidebar">
        <div class="h-full px-3 py-4 overflow-y-auto bg-neutral-primary-soft border-e border-default">
            <ul class="space-y-2 font-medium">
                <li v-for="item in items" :key="item.title">
                    <RouterLink :to="item.to"
                        class="flex items-center px-2 py-1.5 text-body rounded-base hover:bg-neutral-tertiary hover:text-fg-brand group">
                        <component :is="item.icon" class="size-5 transition duration-75 group-hover:text-fg-brand"
                            aria-hidden="true" />
                        <span class="ms-3">{{ item.title }}</span>
                    </RouterLink>
                </li>
            </ul>
        </div>
    </aside>
</template>