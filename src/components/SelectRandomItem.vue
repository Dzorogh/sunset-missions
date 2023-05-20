<template>
  <n-input
    v-model:value="missionsInput"
    type="textarea"
    :autosize="{
        minRows: 4
    }"
  />

  <div>
    Шаг: {{ step }}
  </div>

  <n-data-table style="margin-top: 40px" :data="missions" :columns="columns" :row-key="rowKey"></n-data-table>

  <div>
    <n-button size="large" type="primary" style="margin-top: 40px" @click="selectNextMission">
      Выбрать вариант
    </n-button>

    <n-button size="large" type="primary" style="margin-top: 40px" @click="testMissions">
      Очистить и запустить тест
    </n-button>
  </div>

  <div style="margin-top: 40px">
    Лог выборов:

    <n-data-table style="margin-top: 40px" :data="log" :columns="logColumns"></n-data-table>
  </div>

  <n-button type="primary" tertiary style="margin-top: 40px" @click="clearLog">
    Очистить лог
  </n-button>
</template>

<script setup lang="ts">
import { useStorage } from '@vueuse/core'
import { nextTick, ref, watch } from "vue"
import { DataTableColumn } from "naive-ui"

interface Mission {
  label: string,
  key: number,
  hash: string,
  last_step_used: number,
  uses: number
}

const step = useStorage<number>('step', 1)
const log = useStorage<Mission[]>('log', [])
const missions = useStorage<Mission[]>('missions', [])
const missionsInput = ref<string>(missions.value.map(i => i.label).join("\r\n"))

async function sha256(message) {
  // encode as UTF-8
  const msgBuffer = new TextEncoder().encode(message)

  // hash the message
  const hashBuffer = await crypto.subtle.digest('SHA-256', msgBuffer)

  // convert ArrayBuffer to Array
  const hashArray = Array.from(new Uint8Array(hashBuffer))

  // convert bytes to hex string

  return hashArray.map(b => b.toString(16).padStart(2, '0')).join('')
}

function getRandomInt(max) {
  return Math.floor(Math.random() * max);
}

watch(missionsInput, async () => {
  missions.value = await Promise.all(
    missionsInput
      .value
      .split(/\r?\n/)
      .filter(Boolean)
      .map(async (i, index) => {
        const label = i.trim()

        return {
          label,
          key: index + 1,
          hash: await sha256(label),
          last_step_used: 0,
          uses: 0,
        }
      }),
  )
})

const rowKey = (mission: Mission) => mission.key

const columns = <DataTableColumn<Mission>[]>[
  {
    title: '#',
    key: 'key',
    sorter: 'default',
    sortOrder: 'ascend',
    width: 50,
  },
  {
    title: 'Пункт',
    key: 'label',
  },
  {
    title: 'Ключ',
    key: 'hash',
  },
  {
    title: 'Последнее использование на шаге',
    key: 'last_step_used',
  },
  {
    title: 'Использований',
    key: 'uses',
  },
]

const logColumns = <DataTableColumn<Mission>[]>[
  {
    title: '#',
    key: 'key',
    width: 50,
  },
  {
    title: 'Пункт',
    key: 'label',
  },
  {
    title: 'Последнее использование на шаге',
    key: 'last_step_used',
  },
  {
    title: 'Использований',
    key: 'uses',
  },
]

const selectNextMission = () => {
  const itemsSorted = missions.value.sort((a, b) => {
    return a.last_step_used - b.last_step_used
  })
  const itemsCount = itemsSorted.length * 0.5

  const foundItemIndex = getRandomInt(itemsCount)

  console.log(itemsSorted, itemsCount, foundItemIndex)

  const foundItem = itemsSorted[foundItemIndex];
  const mission = missions.value.find((m) => m.hash === foundItem.hash)

  mission.last_step_used = step.value
  mission.uses++

  log.value.push({...mission})
  step.value++
}

const clearLog = () => {
  step.value = 1
  log.value = []

  missions.value.forEach(mission => {
    mission.uses = 0
    mission.last_step_used = 0
  })
}

const testMissions = () => {
  clearLog()

  nextTick(() => {
    let times = 100

    while (times--) {

      selectNextMission()

    }
  })
}
</script>
