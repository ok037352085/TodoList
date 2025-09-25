<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import DayModal from './DayModal.vue'

const selectedYear = ref()
const selectedMonth = ref()
const showModal = ref(false)
const activeDate = ref(null)
const todos = ref({})

const TODOS_KEY = 'calendarTodos'

onMounted(() => {
  const savedYear = localStorage.getItem('calendarYear')
  const savedMonth = localStorage.getItem('calendarMonth')

  if (savedYear) selectedYear.value = parseInt(savedYear)
  if (savedMonth) selectedMonth.value = parseInt(savedMonth)

  loadTodosFromStorage()
})

watch([selectedYear, selectedMonth], ([year, month]) => {
  localStorage.setItem('calendarYear', year)
  localStorage.setItem('calendarMonth', month)
})

const yearOptions = Array.from({ length: 11 }, (_, i) => 2020 + i)
const monthOptions = Array.from({ length: 12 }, (_, i) => i)

const startDay = computed(() => new Date(selectedYear.value, selectedMonth.value).getDay())
const daysInMonth = computed(() => new Date(selectedYear.value, selectedMonth.value, 0).getDate())

const calendarCells = computed(() => {
  const cells = Array(startDay).fill(null)

  for (let i = 1; i <= daysInMonth.value; i++) {
    const dateObj = new Date(selectedYear.value, selectedMonth.value, i)
    cells.push({
      date: dateObj,
      label: i,
      dayName: ['(日)', '(一)', '(二)', '(三)', '(四)', '(五)', '(六)'][dateObj.getDay()]
    })
  }

  return cells
})

function openModal(date) {
  activeDate.value = date
  showModal.value = true
}

function updateTodos({ date, todos: updatedList }) {
  const key = getDateKey(date)
  todos.value[key] = updatedList
}

function loadTodosFromStorage() {
  try {
    const data = localStorage.getItem(TODOS_KEY)
    if (data) todos.value = JSON.parse(data)
  } catch (e) {
    console.error('讀取待辦資料錯誤', e)
  }
}

function saveTodosToStorage() {
  localStorage.setItem(TODOS_KEY, JSON.stringify(todos.value))
}

watch(todos, saveTodosToStorage, { deep: true })

function getDateKey(date) {
  return date.toISOString().split('T')[0]
}

function isToday(date) {
    const today = new Date()
    return (
        date.getFullYear() === today.getFullYear() &&
        date.getMonth() === today.getMonth() &&
        date.getDate() === today.getDate()
    )
}

function hasTodos(date) {
    const key = getDateKey(date)
    return todos.value[key] && todos.value[key].length > 0
}
</script>

<template>
  <section class="calendar-section">
    <div class="controls">
      <select v-model="selectedYear" class="year-select">
        <option v-for="y in yearOptions" :key="y" :value="y">{{ y }} 年</option>
      </select>
      <select v-model="selectedMonth" class="month-select">
        <option v-for="m in monthOptions" :key="m" :value="m">{{ m + 1 }} 月</option>
      </select>
    </div>

    <div class="calendar-grid">
      <div
        v-for="(cell, index) in calendarCells"
        :key="index"
        class="calendar-cell"
        :class="{
        today: cell && isToday(cell.date),
        hasTodos: cell && !isToday(cell.date) && hasTodos(cell.date)
        }"
        @click="cell && openModal(cell.date)">
        
        <span v-if="cell">{{ cell.label }}</span>
        <span v-if="cell">{{ cell.dayName }}</span>
        <div class="bottom-line" v-if="cell && hasTodos(cell.date)"></div>
    </div>
    </div>

    <DayModal
      :visible="showModal"
      :date="activeDate"
      :todos="todos"
      @close="showModal = false"
      @update-todos="updateTodos"
    />
  </section>
</template>

<style scoped>
.calendar-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 90%;
  max-width: 1200px;
  margin: 0 auto;
  font-family: "Poppins", sans-serif;
}

.controls {
  width: 100%;
  background: #555;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  box-sizing: border-box;
}

.year-select,
.month-select {
  flex: 0 0 auto;
  font-size: 1.5rem;
  background: transparent;
  border: none;
  appearance: none;
  margin: 5px;
  color: #fff;
}

select:focus {
  outline: none;
}

select option {
  color: #000;
}

.calendar-grid {
  width: 100%;
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 2px;
}

.calendar-cell {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  border: 1px solid #ccc;
  min-height: 70px;
  padding: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  word-break: break-word;
}

.calendar-cell.today {
  background: pink;
  color: #fff;
  font-weight: bold;
}

.bottom-line {
  position: absolute;
  width: 100%;
  bottom: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: red;
  border-radius: 2px;
}

@media (max-width: 940px) {
  .calendar-section {
    width: 95%;
  }

  .year-select,
  .month-select {
    font-size: 1.2rem;
  }

  .calendar-cell {
    min-height: 60px;
    font-size: 0.8rem;
    padding: 4px;
  }
}

@media (max-width: 600px) {
  .controls {
    flex-direction: column;
    align-items: center;
    gap: 8px;
  }

  .year-select,
  .month-select {
    font-size: 1rem;
    margin: 0;
  }

  .calendar-grid {
    grid-template-columns: repeat(7, 1fr);
    gap: 1px;
  }

  .calendar-cell {
    min-height: 50px;
    font-size: 0.75rem;
    padding: 2px;
  }
}
</style>

