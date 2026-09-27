<script setup lang="ts">
  import {
    Item,
    ItemContent,
    ItemDescription,
    ItemFooter,
    ItemHeader,
    ItemMedia,
    ItemTitle,
  } from '@/components/ui/item'

  import GlucoseTrendHomepage from '@/components/glicechart_components/GlucoseTrendHomepage.vue'
</script>



<template>



  <Item>
    <ItemHeader>Item Header</ItemHeader>
    <ItemMedia />
    <ItemContent>
      <ItemTitle>Item</ItemTitle>
      <ItemDescription>Item</ItemDescription>
    </ItemContent>
    <ItemFooter>Item Footer</ItemFooter>
  </Item>

  <GlucoseTrendHomepage />


</template>