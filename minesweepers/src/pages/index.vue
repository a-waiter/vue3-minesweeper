<script setup lang="ts">
import type { BlockState } from '~/types'

const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-orange-500',
  'text-red-500',
  'text-purple-500',
  'text-pink-500',
  'text-teal-500'
]

const directions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, 1],
  [0, -1],
  [-1, 1],
  [-1, 0],
  [-1, -1]
]

const WIDTH = 10
const HEIGHT = 10

const state = ref(
  Array.from({ length: HEIGHT }, (_, y) =>
    Array.from({ length: WIDTH }, (_, x): BlockState => {
      return {
        x,
        y,
        revealed: false,
        adjacentMines: 0
      }
    })
  )
)

console.log(state.value)

let mineGenerated = false
const dev = false

function getBlockClass(block: BlockState) {
  if (block.flagged) return 'bg-gray-500/10'
  if (!block.revealed) return 'bg-gray-500/10 hover:bg-gray-500/20'

  return block.mine ? 'bg-red-500/50' : numberColors[block.adjacentMines]
}

function generateMines(initial: BlockState) {
  for (const row of state.value) {
    for (const block of row) {
      if (Math.abs(initial.y - block.x) <= 1) continue
      if (Math.abs(initial.y - block.x) <= 1) continue
      block.mine = Math.random() < 0.2
    }
  }
  updateNumbers()
}

function updateNumbers() {
  state.value.forEach((row, y) => {
    row.forEach((block, x) => {
      if (block.mine) return

      getSiblings(block).forEach((b) => {
        if (b?.mine) block.adjacentMines += 1
      })
    })
  })
}

function expendZero(block: BlockState) {
  if (block.adjacentMines) return

  getSiblings(block).forEach((s) => {
    if (!s.revealed) {
      s.revealed = true
      expendZero(s)
    }
  })
}

function getSiblings(block: BlockState) {
  return directions
    .map(([dx, dy]) => {
      const x = block.x + dx
      const y = block.y + dy

      if (x < 0 || x >= WIDTH || y < 0 || y >= HEIGHT) {
        return undefined
      }
      return state.value[y][x]
    })
    .filter(Boolean) as BlockState[]
}

function onClick(block: BlockState) {
  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }

  if (block.flagged) {
    return
  }

  block.revealed = true
  if (block.mine) {
    revealAllMine()
    alert('BOOOOM!')
  }

  expendZero(block)
}

function onRightClick(block: BlockState) {
  if (block.revealed) return
  block.flagged = !block.flagged
}

function revealAllMine() {
  state.value.forEach((row, y) => {
    row.forEach((block, x) => {
      if (block.mine) block.revealed = true
    })
  })
}

function checkGameState() {
  if (!mineGenerated) return
  const blocks = state.value.flat()
  if (blocks.every((block) => block.revealed || block.flagged)) {
    if (blocks.some((block) => block.flagged && !block.mine))
      alert('You cheat!')
    else alert('You win!')
  }
}

watchEffect(checkGameState)
</script>

<template>
  <div>
    <span fs-12 color-red> minesweepers </span>
    <div p5 w-full overflow-auto>
      <div
        v-for="(row, y) in state"
        :key="y"
        flex="~"
        items-center
        justify-center
        w-max
        ma
      >
        <button
          v-for="(block, x) in row"
          :key="x"
          flex="~"
          items-center
          justify-center
          min-w-8
          min-h-8
          m="3px"
          border="0.5 gray-400/10"
          :class="getBlockClass(block)"
          @click="onClick(block)"
          @contextmenu.prevent="onRightClick(block)"
        >
          <template v-if="block.flagged">
            <div i-mdi-flag text-red />
          </template>
          <template v-else-if="block.revealed || dev">
            <div v-if="block.mine" i-mdi-mine />
            <div v-else font-600>
              {{ block.adjacentMines }}
            </div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>
