<script setup lang="ts">
interface BlockState {
  x: number
  y: number
  revealed: boolean
  mine?: boolean
  flagged?: boolean
  adjacentMines: number
}
const WIDTH = 10
const HEIGHT = 10

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

const state = reactive(
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

let mineGenerated = false
const dev = true

function getBlockClass(block: BlockState) {
  if (block.flagged) return 'bg-gray-500/10'
  if (!block.revealed) return 'bg-gray-500/10 hover:bg-gray-500/20'

  return block.mine ? 'bg-red-500/50' : numberColors[block.adjacentMines]
}

function generateMines(initial: BlockState) {
  for (const row of state) {
    for (const block of row) {
      if (Math.abs(initial.y - block.x) <= 1) continue
      if (Math.abs(initial.y - block.x) <= 1) continue
      block.mine = Math.random() < 0.2
    }
  }
  updateNumbers()
}

function updateNumbers() {
  state.forEach((row, y) => {
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
      return state[y][x]
    })
    .filter(Boolean) as BlockState[]
}

function onClick(block: BlockState) {
  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }

  block.revealed = true
  if (block.mine) alert('BOOOOM!')

  expendZero(block)
}
</script>

<template>
  <div>
    minesweepers
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
          m="1px"
          border="0.5 gray-400/10"
          @click="onClick(block)"
          :class="getBlockClass(block)"
        >
          <template v-if="block.flagged">
            <div i-mdi-flag text-red />
          </template>
          <template v-else-if="block.revealed">
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
