<script setup lang="ts">
import { computed } from 'vue'
import type { HarvestChallengeState, HarvestChallengeProgress } from '@/types/game'
import { HARVEST_CHALLENGE_GOAL_DESCRIPTIONS } from '@/utils/constants'

const props = defineProps<{
  challenge: HarvestChallengeState
  progress: HarvestChallengeProgress
  currentDay: number
}>()

const daysLeft = computed(() => Math.max(0, props.challenge.daysLimit - props.currentDay))

const progressItems = computed(() => {
  return HARVEST_CHALLENGE_GOAL_DESCRIPTIONS.map(desc => {
    const goalValue = props.challenge.goals[desc.key as keyof typeof props.challenge.goals]
    const currentValue = props.progress[desc.key as keyof typeof props.progress] as number

    let achieved = false
    let displayValue = currentValue
    let percentage = 0
    let isPending = false

    if (desc.key === 'survivalRate') {
      if (!props.progress.hasHatchingRecord) {
        isPending = true
        percentage = 0
      } else {
        achieved = currentValue >= goalValue
        percentage = Math.min(100, Math.round((currentValue / goalValue) * 100))
      }
    } else {
      achieved = currentValue >= goalValue
      percentage = Math.min(100, Math.round((currentValue / goalValue) * 100))
    }

    return {
      ...desc,
      goalValue,
      currentValue,
      displayValue,
      achieved,
      percentage,
      isPending,
    }
  })
})

const allAchieved = computed(() => progressItems.value.every(item => item.achieved))
</script>

<template>
  <div class="glass rounded-2xl p-4 border border-amber-400/30 bg-gradient-to-r from-amber-500/10 to-orange-500/10">
    <div class="flex items-center justify-between mb-3">
      <div class="flex items-center gap-2">
        <span class="text-xl">🌾</span>
        <span class="font-display text-amber-300 font-bold">丰收季挑战</span>
      </div>
      <div
        class="px-3 py-1 rounded-full text-xs font-bold"
        :class="daysLeft <= 3 ? 'bg-red-500/30 text-red-300' : 'bg-amber-500/30 text-amber-300'"
      >
        剩余 {{ daysLeft }} 天
      </div>
    </div>

    <div v-if="allAchieved" class="mb-3 px-3 py-2 rounded-xl bg-green-500/20 border border-green-400/30 text-center">
      <span class="text-green-300 text-sm font-bold">🌟 双目标已达成！太棒了！</span>
    </div>

    <div class="grid grid-cols-2 gap-3">
      <div
        v-for="item in progressItems"
        :key="item.key"
        class="bg-white/5 rounded-xl p-3 border transition-all"
        :class="item.achieved ? 'border-green-400/40 bg-green-500/10' : 'border-white/10'"
      >
        <div class="flex items-center justify-between mb-1.5">
          <div class="flex items-center gap-1.5">
            <span>{{ item.emoji }}</span>
            <span class="text-white/80 text-sm font-medium">{{ item.label }}</span>
          </div>
          <span v-if="item.achieved" class="text-green-400">✓</span>
        </div>
        <div class="flex items-baseline gap-1 mb-1.5">
          <template v-if="item.isPending">
            <span class="text-white/50 text-xs font-medium">孵化中...</span>
          </template>
          <template v-else>
            <span
              class="text-lg font-bold"
              :class="item.achieved ? 'text-green-400' : 'text-white'"
            >
              {{ item.displayValue }}{{ item.suffix }}
            </span>
            <span class="text-white/40 text-xs">/ {{ item.goalValue }}{{ item.suffix }}</span>
          </template>
        </div>
        <div class="h-2 bg-black/30 rounded-full overflow-hidden">
          <div
            class="h-full rounded-full transition-all duration-500"
            :class="item.achieved ? 'bg-gradient-to-r from-green-400 to-emerald-500' : 'bg-gradient-to-r from-amber-400 to-orange-500'"
            :style="{ width: `${item.percentage}%` }"
          />
        </div>
      </div>
    </div>
  </div>
</template>
