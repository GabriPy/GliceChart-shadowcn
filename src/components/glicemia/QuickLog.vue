<script setup>
import { ref, computed } from 'vue'
import { Minus, Plus } from '@lucide/vue'
import { Button } from '@/components/ui/button'
import { ToggleGroup, ToggleGroupItem } from '@/components/ui/toggle-group'

const emit = defineEmits(['save'])

const TYPES = {
    rapida: { tab: 'Rapida', name: 'insulina rapida', unit: 'U', chips: [1, 2, 4, 6] },
    lenta: { tab: 'Lenta', name: 'insulina lenta', unit: 'U', chips: [10, 15, 20, 30] },
    carbo: { tab: 'Carboidrati', name: 'carboidrati', unit: 'g', chips: [10, 20, 30, 60] },
}

const type = ref('rapida')
const amount = ref(0)
const cfg = computed(() => TYPES[type.value])
const step = computed(() => (type.value === 'carbo' ? 5 : 1))

const bump = (d) => (amount.value = Math.max(0, (Number(amount.value) || 0) + d))
function setType(v) {
    if (!v) return // ToggleGroup single emette '' se si riclicca la voce attiva
    type.value = v
    amount.value = 0 // le unità cambiano: si riparte da zero
}
function save() {
    emit('save', { type: type.value, value: Number(amount.value), at: Date.now() })
    amount.value = 0
}
</script>

<template>
    <form class="gc-surface space-y-5 rounded-xl p-5" @submit.prevent="save">
        <h2 class="text-base font-medium">Registra adesso</h2>

        <ToggleGroup type="single" variant="outline" class="grid w-full grid-cols-3" :model-value="type"
            @update:model-value="setType">
            <ToggleGroupItem v-for="(c, k) in TYPES" :key="k" :value="k" class="px-2 text-sm">
                {{ c.tab }}
            </ToggleGroupItem>
        </ToggleGroup>

        <div class="flex items-center justify-between gap-2">
            <Button type="button" variant="outline" size="icon" aria-label="Diminuisci" @click="bump(-step)">
                <Minus />
            </Button>
            <label class="flex items-baseline gap-1">
                <input v-model.number="amount" type="number" inputmode="numeric" min="0" aria-label="Quantità"
                    class="gc-num w-24 rounded bg-transparent text-center text-5xl outline-none focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-4 [appearance:textfield] [&::-webkit-inner-spin-button]:appearance-none [&::-webkit-outer-spin-button]:appearance-none" />
                <span class="gc-muted">{{ cfg.unit }}</span>
            </label>
            <Button type="button" variant="outline" size="icon" aria-label="Aumenta" @click="bump(step)">
                <Plus />
            </Button>
        </div>

        <div class="flex flex-wrap gap-2">
            <Button v-for="n in cfg.chips" :key="n" type="button" variant="secondary" size="sm" @click="amount = n">
                {{ n }} {{ cfg.unit }}
            </Button>
        </div>

        <Button type="submit" class="w-full" :disabled="!(amount > 0)">Salva {{ cfg.name }}</Button>
    </form>
</template>