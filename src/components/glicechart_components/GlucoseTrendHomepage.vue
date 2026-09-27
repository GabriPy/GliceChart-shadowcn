<script setup lang="ts">
import { ref, computed } from "vue"
import type { ChartConfig } from "@/components/ui/chart"
import { VisArea, VisAxis, VisLine, VisXYContainer } from "@unovis/vue"
import {
    Card,
    CardContent,
    CardDescription,
    CardHeader,
    CardTitle,
} from "@/components/ui/card"
import {
    ChartContainer,
    ChartCrosshair,
    ChartLegendContent,
    ChartTooltip,
    ChartTooltipContent,
    componentToString,
} from "@/components/ui/chart"
import {
    Select,
    SelectContent,
    SelectItem,
    SelectTrigger,
    SelectValue,
} from "@/components/ui/select"

const description = "An interactive area chart"

const chartData = [
    { date: new Date("2024-04-01"), glicemia: 142 },
    { date: new Date("2024-04-02"), glicemia: 97 },
    { date: new Date("2024-04-03"), glicemia: 167 },
    { date: new Date("2024-04-04"), glicemia: 210 },
    { date: new Date("2024-04-05"), glicemia: 185 },
    { date: new Date("2024-04-06"), glicemia: 124 },
    { date: new Date("2024-04-07"), glicemia: 245 },
    { date: new Date("2024-04-08"), glicemia: 158 },
    { date: new Date("2024-04-09"), glicemia: 68 },
    { date: new Date("2024-04-10"), glicemia: 112 },
    { date: new Date("2024-04-11"), glicemia: 175 },
    { date: new Date("2024-04-12"), glicemia: 130 },
    { date: new Date("2024-04-13"), glicemia: 190 },
    { date: new Date("2024-04-14"), glicemia: 137 },
    { date: new Date("2024-04-15"), glicemia: 120 },
    { date: new Date("2024-04-16"), glicemia: 138 },
    { date: new Date("2024-04-17"), glicemia: 230 },
    { date: new Date("2024-04-18"), glicemia: 164 },
    { date: new Date("2024-04-19"), glicemia: 105 },
    { date: new Date("2024-04-20"), glicemia: 89 },
    { date: new Date("2024-04-21"), glicemia: 137 },
    { date: new Date("2024-04-22"), glicemia: 224 },
    { date: new Date("2024-04-23"), glicemia: 138 },
    { date: new Date("2024-04-24"), glicemia: 187 },
    { date: new Date("2024-04-25"), glicemia: 215 },
    { date: new Date("2024-04-26"), glicemia: 75 },
    { date: new Date("2024-04-27"), glicemia: 183 },
    { date: new Date("2024-04-28"), glicemia: 122 },
    { date: new Date("2024-04-29"), glicemia: 155 },
    { date: new Date("2024-04-30"), glicemia: 254 },
    { date: new Date("2024-05-01"), glicemia: 165 },
    { date: new Date("2024-05-02"), glicemia: 193 },
    { date: new Date("2024-05-03"), glicemia: 147 },
    { date: new Date("2024-05-04"), glicemia: 185 },
    { date: new Date("2024-05-05"), glicemia: 221 },
    { date: new Date("2024-05-06"), glicemia: 198 },
    { date: new Date("2024-05-07"), glicemia: 188 },
    { date: new Date("2024-05-08"), glicemia: 149 },
    { date: new Date("2024-05-09"), glicemia: 227 },
    { date: new Date("2024-05-10"), glicemia: 163 },
    { date: new Date("2024-05-11"), glicemia: 135 },
    { date: new Date("2024-05-12"), glicemia: 197 },
    { date: new Date("2024-05-13"), glicemia: 110 },
    { date: new Date("2024-05-14"), glicemia: 248 },
    { date: new Date("2024-05-15"), glicemia: 173 },
    { date: new Date("2024-05-16"), glicemia: 138 },
    { date: new Date("2024-05-17"), glicemia: 219 },
    { date: new Date("2024-05-18"), glicemia: 115 },
    { date: new Date("2024-05-19"), glicemia: 135 },
    { date: new Date("2024-05-20"), glicemia: 177 },
    { date: new Date("2024-05-21"), glicemia: 82 },
    { date: new Date("2024-05-22"), glicemia: 81 },
    { date: new Date("2024-05-23"), glicemia: 152 },
    { date: new Date("2024-05-24"), glicemia: 194 },
    { date: new Date("2024-05-25"), glicemia: 201 },
    { date: new Date("2024-05-26"), glicemia: 113 },
    { date: new Date("2024-05-27"), glicemia: 220 },
    { date: new Date("2024-05-28"), glicemia: 133 },
    { date: new Date("2024-05-29"), glicemia: 78 },
    { date: new Date("2024-05-30"), glicemia: 140 },
    { date: new Date("2024-05-31"), glicemia: 178 },
    { date: new Date("2024-06-01"), glicemia: 128 },
    { date: new Date("2024-06-02"), glicemia: 270 },
    { date: new Date("2024-06-03"), glicemia: 103 },
    { date: new Date("2024-06-04"), glicemia: 239 },
    { date: new Date("2024-06-05"), glicemia: 88 },
    { date: new Date("2024-06-06"), glicemia: 194 },
    { date: new Date("2024-06-07"), glicemia: 123 },
    { date: new Date("2024-06-08"), glicemia: 185 },
    { date: new Date("2024-06-09"), glicemia: 238 },
    { date: new Date("2024-06-10"), glicemia: 155 },
    { date: new Date("2024-06-11"), glicemia: 92 },
    { date: new Date("2024-06-12"), glicemia: 212 },
    { date: new Date("2024-06-13"), glicemia: 81 },
    { date: new Date("2024-06-14"), glicemia: 226 },
    { date: new Date("2024-06-15"), glicemia: 107 },
    { date: new Date("2024-06-16"), glicemia: 171 },
    { date: new Date("2024-06-17"), glicemia: 275 },
    { date: new Date("2024-06-18"), glicemia: 107 },
    { date: new Date("2024-06-19"), glicemia: 141 },
    { date: new Date("2024-06-20"), glicemia: 208 },
    { date: new Date("2024-06-21"), glicemia: 169 },
    { date: new Date("2024-06-22"), glicemia: 117 },
    { date: new Date("2024-06-23"), glicemia: 280 },
    { date: new Date("2024-06-24"), glicemia: 132 },
    { date: new Date("2024-06-25"), glicemia: 141 },
    { date: new Date("2024-06-26"), glicemia: 234 },
    { date: new Date("2024-06-27"), glicemia: 248 },
    { date: new Date("2024-06-28"), glicemia: 149 },
    { date: new Date("2024-06-29"), glicemia: 103 },
    { date: new Date("2024-06-30"), glicemia: 246 },
]
type Data = typeof chartData[number]

const chartConfig = {
    glicemia: {
        label: "Glicemia",
        color: "var(--chart-2)",
    },
} satisfies ChartConfig

const svgDefs = `
  <linearGradient id="fillDesktop" x1="0" y1="0" x2="0" y2="1">
    <stop
      offset="5%"
      stop-color="var(--chart-1)"
      stop-opacity="0.8"
    />
    <stop
      offset="95%"
      stop-color="var(--chart-1)"
      stop-opacity="0.1"
    />
  </linearGradient>
  <linearGradient id="fillMobile" x1="0" y1="0" x2="0" y2="1">
    <stop
      offset="5%"
      stop-color="var(--chart-2)"
      stop-opacity="0.8"
    />
    <stop
      offset="95%"
      stop-color="var(--chart-2)"
      stop-opacity="0.1"
    />
  </linearGradient>
`

const timeRange = ref("90d")
const filterRange = computed(() => {
    return chartData.filter((item) => {
        const date = new Date(item.date)
        const referenceDate = new Date("2024-06-30")
        let hoursToSubtract = 1
        if (timeRange.value === "1h") {
            hoursToSubtract = 1
        }
        else if (timeRange.value === "3h") {
            hoursToSubtract = 3
        }
        else if (timeRange.value === "9h") {
            hoursToSubtract = 9
        }
        const startDate = new Date(referenceDate)
        startDate.setHours(startDate.getHours() - hoursToSubtract)
        return date >= startDate
    })
})
</script>

<template>


    <div class="grid grid-cols-2 grid-rows-1 gap-2">
        <div>6</div>
        <div>
            <Card class="pt-0">
                <CardHeader class="flex items-center gap-2 space-y-0 border-b py-5 sm:flex-row">
                    <div class="grid flex-1 gap-1">
                        <CardTitle>Area Chart - Interactive</CardTitle>
                        <CardDescription>
                            Showing total visitors for the last 3 months
                        </CardDescription>
                    </div>
                    <Select v-model="timeRange">
                        <SelectTrigger class="hidden w-[160px] rounded-lg sm:ml-auto sm:flex"
                            aria-label="Select a value">
                            <SelectValue placeholder="Ultima ora" />
                        </SelectTrigger>
                        <SelectContent class="rounded-xl">
                            <SelectItem value="1h" class="rounded-lg">
                                Ultima ora
                            </SelectItem>
                            <SelectItem value="3h" class="rounded-lg">
                                Ultime 3 ore
                            </SelectItem>
                            <SelectItem value="9h" class="rounded-lg">
                                Ultime 9 ore
                            </SelectItem>
                        </SelectContent>
                    </Select>
                </CardHeader>
                <CardContent class="px-2 pt-4 sm:px-6 sm:pt-6 pb-4">
                    <ChartContainer :config="chartConfig" class="aspect-auto h-[250px] w-full" :cursor="false">
                        <VisXYContainer :data="filterRange" :svg-defs="svgDefs" :margin="{ left: -40 }"
                            :y-domain="[0, 1200]">
                            <VisArea :x="(d: Data) => d.date" :y="(d: Data) => d.glicemia" color="url(#fillGlicemia)"
                                :opacity="0.6" />
                            <VisLine :x="(d: Data) => d.date" :y="(d: Data) => d.glicemia"
                                :color="chartConfig.glicemia.color" :line-width="2" />
                            <VisAxis type="x" :x="(d: Data) => d.date" :tick-line="false" :domain-line="false"
                                :grid-line="false" :num-ticks="6" :tick-format="(d: number, index: number) => {
                                    const date = new Date(d)
                                    return date.toLocaleDateString('en-US', {
                                        month: 'short',
                                        day: 'numeric',
                                    })
                                }" />
                            <VisAxis type="y" :num-ticks="3" :tick-line="false" :domain-line="false" />
                            <ChartTooltip />
                            <ChartCrosshair :template="componentToString(chartConfig, ChartTooltipContent, {
                                labelFormatter: (d) => {
                                    return new Date(d).toLocaleDateString('en-US', {
                                        month: 'short',
                                        day: 'numeric',
                                    })
                                },
                            })"
                                :color="(d: Data, i: number) => [chartConfig.mobile.color, chartConfig.desktop.color][i % 2]" />
                        </VisXYContainer>

                        <ChartLegendContent />
                    </ChartContainer>
                </CardContent>
            </Card>
        </div>
    </div>



</template>