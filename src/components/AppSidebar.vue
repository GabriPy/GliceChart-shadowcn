<script setup lang="ts">
import { Calendar, Home, Inbox, Search, Settings } from '@lucide/vue' 
import { RouterLink } from 'vue-router'
import {
  Sidebar,
  SidebarContent,
  SidebarGroup,
  SidebarGroupContent,
  SidebarGroupLabel,
  SidebarMenu,
  SidebarMenuButton,
  SidebarMenuItem,
} from '@/components/ui/sidebar'

// 1. Definizione delle voci del menu con la proprietà 'to' al posto di 'url'
const items = [
  {
    title: 'Home',
    to: '/',
    icon: Home,
  },
  {
    title: 'Inbox',
    to: '/inbox',
    icon: Inbox,
  },
  {
    title: 'Calendar',
    to: '/calendar',
    icon: Calendar,
  },
  {
    title: 'Search',
    to: '/search',
    icon: Search,
  },
  {
    title: 'Settings',
    to: '/settings',
    icon: Settings,
  },
]
</script>

<template>
  <Sidebar>
    <SidebarContent>
      <!-- 2. Gruppo principale -->
      <SidebarGroup>
        <SidebarGroupLabel>Application</SidebarGroupLabel>
        <SidebarGroupContent>
          <!-- 3. Lista dinamica del menu -->
          <SidebarMenu>
            <SidebarMenuItem v-for="item in items" :key="item.title">
              <SidebarMenuButton as-child>
                <!-- Usiamo RouterLink al posto di <a> -->
                <RouterLink :to="item.to">
                  <component :is="item.icon" />
                  <span>{{ item.title }}</span>
                </RouterLink>
              </SidebarMenuButton>
            </SidebarMenuItem>
          </SidebarMenu>
        </SidebarGroupContent>
      </SidebarGroup>
    </SidebarContent>
  </Sidebar>
</template>