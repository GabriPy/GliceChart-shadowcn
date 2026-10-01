<script setup>
import { ref, watch, onMounted, onBeforeUnmount } from 'vue'
import {
    Chart, LineController, LineElement, PointElement, LinearScale, TimeScale, Tooltip,
} from 'chart.js'
import 'chartjs-adapter-date-fns'

Chart.register(LineController, LineElement, PointElement, LinearScale, TimeScale, Tooltip)

const props = defineProps({
    readings: { type: Array, required: true }, // [{ t: ms, v: mg/dL }] ordinate per t
    events: { type: Array, default: () => [] }, // [{ t, type: 'rapida' | 'lenta' | 'carbo' | 'nota', value }]
    windowMs: { type: Number, default: 3 * 3600e3 },
    low: { type: Number, default: 70 },
    high: { type: Number, default: 180 },
    gapMs: { type: Number, default: 15 * 60e3 }, // oltre questo intervallo la linea si interrompe
})

const canvas = ref(null)
let chart
let mo
let colors = {}
let state = { gaps: [], events: [] }

const css = (n) => getComputedStyle(document.documentElement).getPropertyValue(n).trim()
const readColors = () => ({
    bg: css('--gc-bg'), surface: css('--gc-surface'), ink: css('--gc-ink'), muted: css('--gc-muted'),
    line: css('--gc-line'), ok: css('--gc-ok'), high: css('--gc-high'), low: css('--gc-low'),
})
const statusColor = (v) => (v < props.low ? colors.low : v > props.high ? colors.high : colors.ok)
const fmt = (t) => new Date(t).toLocaleTimeString('it-IT', { hour: '2-digit', minute: '2-digit' })
const eventLabel = (e) =>
    e.type === 'carbo' ? `${e.value} g` : e.type === 'nota' ? '' : `${e.value} U${e.type === 'lenta' ? ' lenta' : ''}`

// Linea con cambio di colore netto alle soglie (gradiente verticale con stop duri)
function strokeGradient(c) {
    const { ctx, chartArea: a, scales } = c
    if (!a) return colors.ok
    const g = ctx.createLinearGradient(0, a.top, 0, a.bottom)
    const off = (v) => Math.min(1, Math.max(0, (scales.y.getPixelForValue(v) - a.top) / (a.bottom - a.top)))
    g.addColorStop(0, colors.high)
    g.addColorStop(off(props.high), colors.high)
    g.addColorStop(off(props.high), colors.ok)
    g.addColorStop(off(props.low), colors.ok)
    g.addColorStop(off(props.low), colors.low)
    g.addColorStop(1, colors.low)
    return g
}

// Fascia target, buchi nei dati, eventi sotto l'asse e linea verticale al passaggio
const gcPlugin = {
    id: 'gc',
    beforeDatasetsDraw(c) {
        const { ctx, chartArea: a, scales: { x, y } } = c
        ctx.save()
        ctx.globalAlpha = 0.1
        ctx.fillStyle = colors.ok
        const yh = y.getPixelForValue(props.high)
        ctx.fillRect(a.left, yh, a.width, y.getPixelForValue(props.low) - yh)
        ctx.globalAlpha = 0.45
        ctx.fillStyle = colors.line
        for (const [s, e] of state.gaps) {
            const xs = x.getPixelForValue(s)
            ctx.fillRect(xs, a.top, x.getPixelForValue(e) - xs, a.height)
        }
        ctx.restore()
    },
    afterDatasetsDraw(c) {
        const { ctx, chartArea: a, scales: { x } } = c
        const ly = c.height - 14
        ctx.save()
        ctx.font = '11px "Archivo Variable", system-ui, sans-serif'
        ctx.textBaseline = 'middle'
        for (const e of state.events) {
            const px = x.getPixelForValue(e.t)
            ctx.strokeStyle = colors.ink
            ctx.fillStyle = colors.ink
            ctx.globalAlpha = 0.35
            ctx.lineWidth = 1
            ctx.setLineDash([1, 4])
            ctx.beginPath()
            ctx.moveTo(px, a.top)
            ctx.lineTo(px, ly - 8)
            ctx.stroke()
            ctx.setLineDash([])
            ctx.globalAlpha = 1
            ctx.lineWidth = 1.5
            ctx.beginPath()
            if (e.type === 'carbo') ctx.arc(px, ly, 4.5, 0, Math.PI * 2)
            else if (e.type === 'nota') ctx.rect(px - 4, ly - 4, 8, 8)
            else {
                ctx.moveTo(px, ly - 5); ctx.lineTo(px + 5, ly); ctx.lineTo(px, ly + 5); ctx.lineTo(px - 5, ly)
                ctx.closePath()
            }
            if (e.type === 'rapida' || e.type === 'carbo') ctx.fill()
            else ctx.stroke()
            ctx.fillStyle = colors.muted
            ctx.fillText(eventLabel(e), px + 9, ly)
        }
        const act = c.tooltip?.getActiveElements?.() ?? []
        if (act.length) {
            ctx.globalAlpha = 0.4
            ctx.strokeStyle = colors.ink
            ctx.setLineDash([])
            ctx.beginPath()
            ctx.moveTo(act[0].element.x, a.top)
            ctx.lineTo(act[0].element.x, a.bottom)
            ctx.stroke()
        }
        ctx.restore()
    },
}

// Ricalcola dati, assi e colori dalle props: è l'unico punto che tocca il grafico
function apply() {
    if (!chart) return
    colors = readColors()
    const t1 = props.readings.at(-1)?.t ?? Date.now()
    const t0 = t1 - props.windowMs
    const pts = props.readings.filter((r) => r.t >= t0)

    const data = []
    const gaps = []
    pts.forEach((p, i) => {
        const prev = pts[i - 1]
        if (prev && p.t - prev.t > props.gapMs) {
            gaps.push([prev.t, p.t])
            data.push({ x: prev.t + 1, y: null }) // il null interrompe la linea
        }
        data.push({ x: p.t, y: p.v })
    })

    const H = 3600e3
    const [unit, stepSize] =
        props.windowMs <= H ? ['minute', 10] : props.windowMs <= 3 * H ? ['minute', 30] : props.windowMs <= 6 * H ? ['hour', 1] : ['hour', 4]
    const vs = pts.map((p) => p.v)
    const o = chart.options

    o.scales.x.min = t0
    o.scales.x.max = t1
    o.scales.x.time.unit = unit
    o.scales.x.time.stepSize = stepSize
    o.scales.x.ticks.color = colors.muted
    o.scales.y.min = Math.floor((Math.min(40, ...vs) - 5) / 10) * 10
    o.scales.y.max = Math.ceil(Math.max(250, ...vs) / 50) * 50
    o.scales.y.ticks.color = colors.muted
    o.scales.y.grid.color = colors.line
    Object.assign(o.plugins.tooltip, {
        backgroundColor: colors.surface, titleColor: colors.muted, bodyColor: colors.ink, borderColor: colors.line,
    })

    state = { gaps, events: props.events.filter((e) => e.t >= t0 && e.t <= t1) }
    chart.data.datasets[0].data = data
    chart.update('none')
}

onMounted(() => {
    colors = readColors()
    Chart.defaults.font.family = '"Archivo Variable", system-ui, sans-serif'
    Chart.defaults.font.size = 11

    chart = new Chart(canvas.value, {
        type: 'line',
        plugins: [gcPlugin],
        data: {
            datasets: [{
                parsing: false,
                normalized: true,
                spanGaps: false,
                borderWidth: 2.25,
                borderColor: (c) => strokeGradient(c.chart),
                pointRadius: (c) => (c.dataIndex === c.dataset.data.length - 1 ? 4.5 : 0),
                pointHoverRadius: 4,
                pointBackgroundColor: (c) => statusColor(c.raw?.y),
                pointBorderColor: () => colors.bg,
                pointBorderWidth: 2,
                data: [],
            }],
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            animation: false,
            layout: { padding: { bottom: 28, right: 8 } },
            interaction: { mode: 'nearest', axis: 'x', intersect: false },
            scales: {
                x: {
                    type: 'time',
                    time: {},
                    grid: { display: false },
                    border: { display: false },
                    ticks: { maxRotation: 0 },
                },
                y: {
                    position: 'left',
                    border: { display: false },
                    grid: {},
                    ticks: { padding: 6 },
                    afterBuildTicks(axis) {
                        axis.ticks = [props.low, props.high, 250, 300]
                            .filter((v) => v >= axis.min && v <= axis.max)
                            .map((value) => ({ value }))
                    },
                },
            },
            plugins: {
                tooltip: {
                    displayColors: false,
                    borderWidth: 1,
                    padding: 8,
                    callbacks: {
                        title: (i) => fmt(i[0].parsed.x),
                        label: (i) => `${i.parsed.y} mg/dL`,
                    },
                },
            },
        },
    })

    apply()
    // Se cambi tema (classe "dark" su <html>) il grafico rilegge i colori
    mo = new MutationObserver(apply)
    mo.observe(document.documentElement, { attributes: true, attributeFilter: ['class'] })
})

onBeforeUnmount(() => {
    mo?.disconnect()
    chart?.destroy()
})

// Qualsiasi cambio dei dati (sostituzione dell'array o push) aggiorna il grafico
watch(
    () => [props.readings, props.readings.length, props.events, props.events.length, props.windowMs, props.low, props.high],
    apply,
)
</script>

<template>
    <div class="relative h-72 w-full select-none sm:h-96">
        <canvas ref="canvas" role="img" aria-label="Andamento glicemico" style="touch-action: pan-y" />
    </div>
</template>