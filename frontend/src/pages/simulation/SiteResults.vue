<template>
  <div
    class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-3 items-center"
  >
    <div>
      <PlotlyDiagram
        v-if="procCapacity"
        title="Processes"
        :data="procCapacity"
        :bargap="0.6"
        titleX="MW"
        :margin="{
          t: 150,
          l: 100,
        }"
      />
    </div>
    <div class="grid grid-cols-2 xl:grid-cols-3 gap-3 justify-items-center">
      <DataPoint
        v-for="proc in newProcs"
        :key="proc.name"
        :name="proc.name"
        :value="proc.value"
        suffix="MW"
      />
      <DataPoint
        v-for="stor in newStor"
        :key="stor.name"
        :name="stor.name"
        :value="stor.value"
        :suffix="stor.suffix"
      />
      <DataPoint
        name="Storage cycles per year"
        :value="
          storRetrieved / newStor.map(s => s.value).reduce((a, b) => a + b, 0)
        "
        suffix=""
      />
    </div>
    <div>
      <PlotlyDiagram
        v-if="storCapPow"
        title="Storage"
        :data="storCapPow"
        :bargap="0.6"
        :titleX="storUnitC?.join(' / ') || 'MWh'"
        :titleX2="storUnitR?.join(' / ') || 'MW'"
        :margin="{
          t: 150,
          l: 150,
        }"
      />
    </div>
    <FloatLabel variant="on" class="col-span-1">
      <Select
        fluid
        id="groupoptions"
        :options="groupOptions"
        optionLabel="name"
        v-model="groupOption"
      />
      <label for="groupoptions">Group values</label>
    </FloatLabel>
    <div v-if="commodityDetails" class="md:col-span-2 xl:col-span-3">
      <PlotlyDiagram
        v-for="(data, comName) in commodityDetails"
        :key="comName"
        :title="<string>comName"
        :data="data.data"
        :titleY="data.unitC"
        barmode="relative"
        :bargroupgap="0.2"
        :margin="{
          t: 150,
        }"
      />
    </div>

    <div class="md:col-span-2 xl:grid-cols-3 grid grid-cols-1 gap-3">
      <FloatLabel variant="on" class="w-full md:w-1/2 xl:w-1/3">
        <Select
          fluid
          id="commodityoptions"
          :options="demandCommodityOptions"
          v-model="selectedCommodity"
        />
        <label for="commodityoptions">Select Commodity for Production View</label>
      </FloatLabel>
      <div v-if="selectedCommodity" class="w-full md:col-span-2">
        <PlotlyDiagram
          v-if="annualProductionData && annualProductionData.length > 0"
          :title="productionChartTitle"
          :data="annualProductionData"
          titleX="GWh"
          :bargap="0.5"
          :xaxis-range="annualProductionXRange"
          :margin="{
            t: 150,
            l: 150,
            r: 50,
            b: 50,
          }"
        />
        <div
          v-else
          class="flex items-center justify-center p-8 text-gray-500"
        >
          <p>No production data available for {{ selectedCommodity }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { Site } from '@/backend/interfaces'
import { useRoute } from 'vue-router'
import { inject, type Ref, ref, watch, computed } from 'vue'
import { useGetSimulation, useGetSimulationConfig } from '@/backend/simulate'
import PlotlyDiagram from '@/plotly/PlotlyDiagram.vue'
import DataPoint from '@/pages/simulation/DataPoint.vue'
import { chunkAdd, groupOptions } from '@/helper/diagrams'

const route = useRoute()

const props = defineProps<{
  site: Site
  selectedCommodityType?: string
  selectedDisplayUnit?: string
}>()

const advanced = inject<Ref<boolean>>('advanced')

const { data: simulation } = useGetSimulation(route)
const { data: config } = useGetSimulationConfig(route)

const procCapacity = ref<Partial<Plotly.Data>[]>()
const newProcs = ref<{ name: string; value: number }[]>([])

const storCapPow = ref<Partial<Plotly.Data>[]>()
const storUnitR = ref<string[]>()
const storUnitC = ref<string[]>()
const newStor = ref<{ name: string; value: number; suffix: string }[]>([])
const storRetrieved = ref(0)

const commodityDetails = ref<{
  [key: string]: { unitR: string; unitC: string; data: Partial<Plotly.Data>[] }
}>({})

const groupOption = ref(groupOptions[0])

// Refs for the annual production chart
const demandCommodityOptions = ref<string[]>([])
const selectedCommodity = ref<string>()
const annualProductionData = ref<Partial<Plotly.Data>[]>()
const annualProductionXRange = ref<[number, number] | undefined>()

const productionChartTitle = computed(() => {
  const com = selectedCommodity.value || 'commodity'
  const rc = config.value?.run_config
  if (
    rc &&
    (rc.timestep_from !== undefined || rc.timestep_to !== undefined)
  ) {
    const from = rc.timestep_from ?? 0
    const to = rc.timestep_to ?? (config.value?.c_timesteps ?? 1) - 1
    return `Production for ${com} (timesteps ${from}–${to})`
  }
  return `Annual Production for ${com}`
})

watch(
  [simulation, advanced, groupOption, selectedCommodity],
  () => {
    // Resetting values on each change
    newProcs.value = []
    procCapacity.value = []
    newStor.value = []
    storCapPow.value = []
    storUnitR.value = []
    storUnitC.value = []
    commodityDetails.value = {}
    annualProductionData.value = undefined
    demandCommodityOptions.value = []
    storRetrieved.value = 0


    if (!simulation.value || !config.value) return

    const siteConfig = config.value.site[props.site.name]
    const siteResults = simulation.value.result

    // Init processes
    {
      const procsNew = []
      const procsInstalled = []
      const procsNames = []
      for (const procName in siteResults.process[props.site.name]) {
        const procConf = siteConfig.process[procName]
        let allIn = true
        let allOut = true
        for (const procConfName in procConf['commodity']) {
          const dir = procConf['commodity'][procConfName].Direction
          allIn &&= dir === 'In'
          allOut &&= dir === 'Out'
        }
        if (!advanced?.value && (allIn || allOut)) continue

        const proc = siteResults.process[props.site.name][procName]
        procsNew.push(proc.New)
        procsInstalled.push(proc.Total - proc.New)
        procsNames.push(procName)

        newProcs.value.push({ name: procName, value: proc.Total })
      }
      procCapacity.value = [
        {
          name: 'Installed Capacity',
          x: procsInstalled,
          y: procsNames,
          type: 'bar',
          orientation: 'h',
        },
        {
          name: 'New Capacity',
          x: procsNew,
          y: procsNames,
          type: 'bar',
          orientation: 'h',
        },
      ]
    }

    // Init storage
    {
      const storCNew = []
      const storCInstalled = []
      const storPNew = []
      const storPInstalled = []
      const storNames = []
      for (const comName in siteResults.storage[props.site.name]) {
        const storCom = siteResults.storage[props.site.name][comName]
        for (const storName in storCom) {
          const stor = storCom[storName]

          storCNew.push(stor.CNew)
          storCInstalled.push(stor.CTotal - stor.CNew)
          storPNew.push(stor.PNew)
          storPInstalled.push(stor.PTotal - stor.PNew)
          storNames.push(storName)

          const com = siteConfig.commodity[comName]
          newStor.value.push({
            name: storName,
            value: stor.CTotal,
            suffix: com.unitC,
          })
          storUnitR.value.push(com.unitR)
          storUnitC.value.push(com.unitC)
        }
      }
      storCapPow.value = [
        {
          name: 'Installed Capacity',
          x: storCInstalled,
          y: storNames,
          type: 'bar',
          xaxis: 'x1',
          orientation: 'h',
        },
        {
          name: 'New Capacity',
          x: storCNew,
          y: storNames,
          type: 'bar',
          xaxis: 'x1',
          orientation: 'h',
        },
        {
          name: 'Installed Power',
          x: storPInstalled,
          y: storNames,
          type: 'scatter',
          mode: 'markers',
          marker: {
            symbol: 25,
            size: 15,
          },
          xaxis: 'x2',
          orientation: 'h',
        },
        {
          name: 'New Power',
          x: storPNew,
          y: storNames,
          type: 'scatter',
          mode: 'markers',
          marker: {
            symbol: 25,
            size: 15,
          },
          xaxis: 'x2',
          orientation: 'h',
        },
      ]
    }

    // Init commodity details (timeseries)
    const results = siteResults.results[props.site.name]
    const timeline = Array.from(
      { length: groupOption.value.groups },
      (_, i) => i + 1,
    )
    const hasFlow = (series?: number[]) =>
      Array.isArray(series) && series.some(v => Math.abs(v) > 1e-9)

    // Iterate the site's *configured* commodities so that every demand
    // commodity gets a chart, even when no timeseries was uploaded/simulated.
    for (const comName in siteConfig.commodity) {
      const com = siteConfig.commodity[comName]

      // Only build a timeseries chart for demand commodities of this site.
      if (com.Type !== 'Demand') continue

      // Populate the dropdown options for the annual production chart.
      demandCommodityOptions.value.push(comName)

      // Result data may be missing entirely when nothing was simulated for it.
      const comResults = results?.[comName]

      // Track total retrieved energy for the "Storage cycles per year" metric.
      storRetrieved.value += (comResults?.storage?.Retrieved ?? []).reduce(
        (acc, num) => acc + num,
        0,
      )

      const traces: Partial<Plotly.Data>[] = []

      // Production per process (positive bars); skip all-zero series.
      for (const procName in comResults?.created ?? {}) {
        const series = chunkAdd(
          comResults!.created[procName],
          groupOption.value.groupSize,
        )
        if (!hasFlow(series)) continue
        traces.push({
          name: procName,
          x: timeline,
          y: series,
          type: 'bar',
          yaxis: 'y1',
        })
      }

      // Storage retrieved (positive) and stored (negative); no Level.
      const retrieved = chunkAdd(
        comResults?.storage?.Retrieved ?? [],
        groupOption.value.groupSize,
      )
      if (hasFlow(retrieved)) {
        traces.push({
          name: 'Storage retrieved',
          x: timeline,
          y: retrieved,
          type: 'bar',
          yaxis: 'y1',
        })
      }
      const stored = chunkAdd(
        comResults?.storage?.Stored ?? [],
        groupOption.value.groupSize,
        -1,
      )
      if (hasFlow(stored)) {
        traces.push({
          name: 'Storage stored',
          x: timeline,
          y: stored,
          type: 'bar',
          yaxis: 'y1',
        })
      }

      // Import from neighbouring sites (positive bars); skip all-zero series.
      for (const sourceSite in comResults?.imported ?? {}) {
        const series = chunkAdd(
          comResults!.imported![sourceSite],
          groupOption.value.groupSize,
        )
        if (!hasFlow(series)) continue
        traces.push({
          name: `Import from ${sourceSite}`,
          x: timeline,
          y: series,
          type: 'bar',
          yaxis: 'y1',
        })
      }

      // Export to neighbouring sites (negative bars); skip all-zero series.
      for (const destSite in comResults?.exported ?? {}) {
        const series = chunkAdd(
          comResults!.exported![destSite],
          groupOption.value.groupSize,
          -1,
        )
        if (!hasFlow(series)) continue
        traces.push({
          name: `Export to ${destSite}`,
          x: timeline,
          y: series,
          type: 'bar',
          yaxis: 'y1',
        })
      }

      // Demand line. Always shown for a demand commodity so the plot exists
      // even when no demand timeseries was uploaded (flat zero in that case).
      const demandRaw = comResults?.demand ?? []
      const demand = demandRaw.length
        ? chunkAdd(demandRaw, groupOption.value.groupSize)
        : new Array(groupOption.value.groups).fill(0)
      traces.push({
        name: 'Demand',
        x: timeline,
        y: demand,
        type: 'scatter',
        yaxis: 'y1',
      })

      commodityDetails.value[comName] = {
        unitR: com.unitR,
        unitC: com.unitC,
        data: traces,
      }
    }

    // Logic for Annual Production Chart
    {
      // Validate and set selected commodity
      if (demandCommodityOptions.value.length > 0) {
        // If no commodity is selected, or the selected one is no longer valid, set to first available
        if (!selectedCommodity.value || !demandCommodityOptions.value.includes(selectedCommodity.value)) {
          selectedCommodity.value = demandCommodityOptions.value[0]
        }
      } else {
        // No demand commodities available, clear selection
        selectedCommodity.value = undefined
      }

      // Reset annual production data first
      annualProductionData.value = undefined
      annualProductionXRange.value = undefined

      if (selectedCommodity.value && results && results[selectedCommodity.value]) {
        const commodityResult = results[selectedCommodity.value]
        
        // Check if created property exists
        if (commodityResult && commodityResult.created) {
          const productionByProcess = commodityResult.created

          // Check if created data exists and is not empty
          if (productionByProcess && typeof productionByProcess === 'object' && !Array.isArray(productionByProcess)) {
            const processNames: string[] = []
            const productionTotals: number[] = []

            for (const procName in productionByProcess) {
              if (productionByProcess.hasOwnProperty(procName)) {
                const processProduction = productionByProcess[procName]
                // Ensure it's an array and has values
                if (Array.isArray(processProduction) && processProduction.length > 0) {
                  const yearlyProductionMWh = processProduction.reduce(
                    (sum, current) => sum + (current || 0),
                    0,
                  )
                  // Convert from MWh to GWh
                  const yearlyProductionGWh = yearlyProductionMWh / 1000

                  // Only add processes with non-zero production
                  if (yearlyProductionGWh > 0) {
                    processNames.push(procName)
                    productionTotals.push(yearlyProductionGWh)
                  }
                }
              }
            }

            if (processNames.length > 0) {
              // Calculate x-axis range to make bars clearly visible
              const maxValue = Math.max(...productionTotals)
              const minValue = 0
              
              // Ensure bars take up a significant portion of the chart
              // Use a multiplier that ensures bars are clearly visible
              // If maxValue is very small, scale it up more aggressively
              let multiplier = 1.5 // Default: bars take up ~67% of chart
              
              if (maxValue < 0.1) {
                multiplier = 3.0 // For very small values, scale up more
              } else if (maxValue < 1) {
                multiplier = 2.0 // For small values, scale up
              }
              
              const rangeMax = maxValue * multiplier
              
              annualProductionXRange.value = [minValue, rangeMax]
              
              console.log('Annual Production Data:', {
                processNames,
                productionTotals,
                selectedCommodity: selectedCommodity.value,
                xRange: annualProductionXRange.value
              })
              
              annualProductionData.value = [
                {
                  name: 'Annual Production',
                  x: productionTotals,
                  y: processNames,
                  type: 'bar',
                  orientation: 'h',
                },
              ]
            } else {
              annualProductionXRange.value = undefined
              console.log('No processes found with production for:', selectedCommodity.value, {
                productionByProcess,
                commodityResult
              })
            }
          }
        }
      }
    }
  },
  {
    immediate: true,
  },
)
</script>

<style scoped></style>
