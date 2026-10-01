<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'
import { ArrowRight } from '@lucide/vue'
import { ToggleGroup, ToggleGroupItem } from '@/components/ui/toggle-group'
import GlucoseChart from '@/components/glicemia/GlucoseChart.vue'
import QuickLog from '@/components/glicemia/QuickLog.vue'

const LOW = 70
const HIGH = 180

// --- Dati demo: sostituisci readings/events con quelli del tuo store ---
// Quando avrai l'endpoint basta assegnare readings.value / events.value:
// grafico, valore attuale e statistiche si aggiornano da soli.
const END = Date.now()
const readings = ref(
  Array.from({ length: 288 }, (_, i) => ({
    t: END - (287 - i) * 5 * 60e3,
    v: Math.round(150 + 90 * Math.sin(i / 22) + 15 * Math.sin(i / 3)),
  })),
)
const events = ref([
  { t: END - 8 * 3600e3, type: 'lenta', value: 14 },
  { t: END - 95 * 60e3, type: 'carbo', value: 30 },
  { t: END - 90 * 60e3, type: 'rapida', value: 4 },
])
function onSave(entry) {
  // entry = { type: 'rapida' | 'lenta' | 'carbo', value, at }
  console.log('salva', entry)
}
// -----------------------------------------------------------------------

const range = ref('3')
const windowMs = computed(() => Number(range.value) * 3600e3)
const rangeLabel = computed(() => (range.value === '1' ? 'Ultima ora' : `Ultime ${range.value} ore`))

const now = ref(Date.now())
let timer
onMounted(() => (timer = setInterval(() => (now.value = Date.now()), 30e3)))
onBeforeUnmount(() => clearInterval(timer))

const last = computed(() => readings.value.at(-1))
const mins = computed(() => Math.max(0, Math.floor((now.value - last.value.t) / 60e3)))
const ago = computed(() =>
  mins.value < 1 ? 'adesso' : mins.value < 60 ? `${mins.value} min fa` : `${Math.floor(mins.value / 60)} h fa`,
)
const stale = computed(() => mins.value > 15)

const status = computed(() => {
  const v = last.value.v
  if (v < LOW) return { label: 'Bassa', color: 'var(--gc-low)' }
  if (v > HIGH) return { label: 'Alta', color: 'var(--gc-high)' }
  return { label: 'In target', color: 'var(--gc-ok)' }
})

// L'inclinazione della freccia segue la velocità di variazione degli ultimi 15 minuti
const trend = computed(() => {
  const l = last.value
  const recent = readings.value.filter((r) => r.t >= l.t - 15 * 60e3)
  const a = recent[0]
  const slope = recent.length > 1 ? (l.v - a.v) / ((l.t - a.t) / 60e3) : 0 // mg/dL al minuto
  if (slope > 2) return { deg: -90, label: 'in forte salita' }
  if (slope > 1) return { deg: -45, label: 'in salita' }
  if (slope >= -1) return { deg: 0, label: 'stabile' }
  if (slope >= -2) return { deg: 45, label: 'in calo' }
  return { deg: 90, label: 'in forte calo' }
})

const stats = computed(() => {
  const vs = readings.value.filter((r) => r.t >= last.value.t - windowMs.value).map((r) => r.v)
  const pct = (f) => Math.round((vs.filter(f).length / vs.length) * 100)
  const low = pct((v) => v < LOW)
  const high = pct((v) => v > HIGH)
  return {
    low,
    high,
    ok: 100 - low - high,
    avg: Math.round(vs.reduce((a, b) => a + b, 0) / vs.length),
    min: Math.min(...vs),
    max: Math.max(...vs),
  }
})
const items = computed(() => {
  const s = stats.value
  return [
    { k: 'In target', v: `${s.ok}%`, c: 'var(--gc-ok)' },
    { k: 'Alta', v: `${s.high}%`, c: 'var(--gc-high)' },
    { k: 'Bassa', v: `${s.low}%`, c: 'var(--gc-low)' },
    { k: 'Media', v: s.avg },
    { k: 'Minima', v: s.min },
    { k: 'Massima', v: s.max },
  ]
})
</script>

<template>
  <main class="gc-root min-h-dvh">
    <div class="mx-auto grid max-w-6xl gap-x-12 gap-y-8 px-4 py-6 sm:px-6 lg:grid-cols-[minmax(0,1fr)_20rem] lg:py-10">
      <section class="min-w-0 space-y-8">
        <!-- valore attuale -->
        <header class="flex flex-wrap items-end justify-between gap-x-6 gap-y-4">
          <div>
            <p class="text-sm" :class="stale ? '' : 'gc-muted'" :style="stale ? { color: 'var(--gc-high)' } : null">
              {{ stale ? `Nessun dato da ${ago.replace(' fa', '')}` : `Ultima lettura ${ago}` }}
            </p>
            <div class="mt-1 flex items-center gap-3" :style="{ color: status.color }">
              <span class="gc-num text-8xl leading-none sm:text-9xl">{{ last.v }}</span>
              <div class="flex flex-col items-start gap-1">
                <ArrowRight class="size-9 shrink-0" :style="{ transform: `rotate(${trend.deg}deg)` }"
                  aria-hidden="true" />
                <span class="gc-muted text-sm">mg/dL</span>
              </div>
            </div>
            <p class="mt-2 text-lg">{{ status.label }}, {{ trend.label }}</p>
          </div>

          <ToggleGroup type="single" variant="outline" size="sm" :model-value="range"
            @update:model-value="(v) => v && (range = v)">
            <ToggleGroupItem v-for="r in ['1', '3', '6', '24']" :key="r" :value="r" :aria-label="`${r} ore`">
              {{ r }} h
            </ToggleGroupItem>
          </ToggleGroup>
        </header>

        <GlucoseChart :readings="readings" :events="events" :window-ms="windowMs" :low="LOW" :high="HIGH" />

        <!-- distribuzione e statistiche -->
        <section class="gc-rule space-y-4 border-t pt-6">
          <div class="flex flex-wrap items-baseline justify-between gap-x-4">
            <h2 class="text-base font-medium">{{ rangeLabel }}</h2>
            <p class="gc-muted text-sm">Target {{ LOW }}–{{ HIGH }} mg/dL</p>
          </div>

          <div class="flex h-2.5 overflow-hidden rounded-full" role="img" :aria-label="`In target ${stats.ok}%`">
            <div :style="{ width: stats.low + '%', background: 'var(--gc-low)' }" />
            <div :style="{ width: stats.ok + '%', background: 'var(--gc-ok)' }" />
            <div :style="{ width: stats.high + '%', background: 'var(--gc-high)' }" />
          </div>

          <dl class="grid grid-cols-3 gap-y-5 sm:grid-cols-6">
            <div v-for="i in items" :key="i.k">
              <dt class="gc-muted flex items-center gap-1.5 text-sm">
                <span v-if="i.c" class="size-2 rounded-full" :style="{ background: i.c }" />
                {{ i.k }}
              </dt>
              <dd class="gc-num text-3xl">{{ i.v }}</dd>
            </div>
          </dl>
        </section>
      </section>

      <aside class="lg:sticky lg:top-10 lg:self-start">
        <QuickLog @save="onSave" />
      </aside>
    </div>
  </main>
</template>