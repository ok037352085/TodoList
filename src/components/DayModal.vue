<script setup>
    import { computed, ref, onMounted, watch, nextTick } from 'vue'

const props = defineProps({
    visible: Boolean,
    date: Date,
    todos: Object,
})

const emit = defineEmits(['close', 'update-todos'])

const newTodo = ref('')

const displayDate = computed(() => {
    if (!props.date) return ''
    const dayNames = ['日', '一', '二', '三', '四', '五', '六']
    const d = props.date
    return `${d.getFullYear()}/${d.getMonth() + 1}/${d.getDate()}（${dayNames[d.getDay()]}）`
})

const dateKey = computed(() =>
    props.date ? props.date.toISOString().split('T')[0] : ''
)
const todosForDate = computed(() => props.todos[dateKey.value] || [])

function addTodo() {
    const trimmed = newTodo.value.trim()
    if (!trimmed) return

    const updated = [...todosForDate.value, trimmed]
    emit('update-todos', { date: props.date, todos: updated })
    newTodo.value = ''
    nextTick(() => todoInput.value?.focus())
}

function removeTodo(index) {
    const updated = [...todosForDate.value]
    updated.splice(index, 1)
    emit('update-todos', { date: props.date, todos: updated })
}

function handleClose() {
    newTodo.value = ''
    emit('close')
}

const todoInput = ref(null)

watch(() => props.visible, (val) => {
  if (val) {
    nextTick(() => todoInput.value?.focus())
  }
})

</script>

<template>
    <div v-if="visible" class="dialog-overlay">
        <div class="dialog-overlay-close" @click="handleClose"></div>
        <div class="dialog-box">
            <h3>{{ displayDate }}</h3>
            <ul class="todo-list">
                <li v-for="(todo, index) in todosForDate" :key="index">
                {{ todo }}
                <button class="delete-btn" @click="removeTodo(index)">X</button>
                </li>
                <li v-if="todosForDate.length === 0" class="empty">（尚無待辦事項）</li>
            </ul>

            <input ref="todoInput" v-model="newTodo" placeholder="新增待辦事項" @keyup.enter="addTodo" />
            <div class="actions">
                <button @click="addTodo">新增</button>
                <button @click="handleClose">關閉</button>
            </div>
        </div>
    </div>
</template>

<style scoped>
    .dialog-overlay {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background: rgba(0, 0, 0, 0.5);
        display: flex;
        align-items: center;
        justify-content: center;
        backdrop-filter: blur(5px);
    }
    .dialog-overlay-close {
        position: fixed;
        width: 100vw;
        height: 100vh;
        z-index: 999;
    }
    .dialog-box {
        background: white;
        padding: 1.5rem;
        border-radius: 8px;
        width: 320px;
        z-index: 1000;
    }
    .todo-list {
        margin: 1rem 0;
        padding: 0;
        list-style: none;
    }
    .todo-list li {
        display: flex;
        justify-content: space-between;
        margin-bottom: 0.5rem;
    }
    .delete-btn {
        background: transparent;
        border: none;
        cursor: pointer;
        color: red;
    }
    .empty {
        color: gray;
        font-style: italic;
    }
    .actions {
        display: flex;
        justify-content: space-between;
        margin-top: 1rem;
    }
    input {
        width: 100%;
        padding: 0.5rem;
        margin-top: 0.5rem;
        box-sizing: border-box;
    }
</style>
