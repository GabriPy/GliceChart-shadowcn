<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
    size?: string | number
}>()

const heightValue = computed(() => {
    if (!props.size) return '1rem'
    return typeof props.size === 'number' || !isNaN(Number(props.size))
        ? `${props.size}px`
        : props.size
})
</script>

<template>
    <div :style="{ height: heightValue }" class="w-full shrink-0" aria-hidden="true" />
</template>