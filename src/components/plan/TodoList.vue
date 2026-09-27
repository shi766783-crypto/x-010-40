<script setup>
import { computed, ref } from 'vue'
import { useTravelStore } from '../../stores/travel'
import { planTodoProgress } from '../../services/selectors'
import ProgressBar from '../common/ProgressBar.vue'
import Modal from '../common/Modal.vue'

const props = defineProps({
  planId: { type: String, required: true },
})

const store = useTravelStore()
const plan = computed(() => store.planById(props.planId))
const todos = computed(() => plan.value?.todos || [])
const progress = computed(() => planTodoProgress(plan.value))

const doneCount = computed(() => todos.value.filter((t) => t.done).length)
const hasUnfinished = computed(() => todos.value.some((t) => !t.done))
const hasCompleted = computed(() => doneCount.value > 0)

const showClearConfirm = ref(false)

const newName = ref('')

function add() {
  const name = newName.value.trim()
  if (!name) return
  store.addTodo(props.planId, name)
  newName.value = ''
}

function completeAll() {
  store.completeAllTodos(props.planId)
}

function confirmClearCompleted() {
  store.clearCompletedTodos(props.planId)
  showClearConfirm.value = false
}
</script>

<template>
  <div>
    <div class="todo-add">
      <input
        v-model="newName"
        class="input"
        placeholder="添加待办事项"
        @keyup.enter="add"
      />
      <button type="button" class="btn btn-primary btn-sm" @click="add">添加</button>
    </div>

    <ProgressBar :value="progress" class="mt-16">
      <template #label><span>待办完成进度</span></template>
    </ProgressBar>

    <div v-if="todos.length" class="todo-bulk mt-16">
      <span class="todo-count">共 {{ todos.length }} 项，已完成 {{ doneCount }} 项</span>
      <div class="todo-bulk-actions">
        <button
          type="button"
          class="btn btn-ghost btn-sm"
          :disabled="!hasUnfinished"
          @click="completeAll"
        >一键完成全部</button>
        <button
          type="button"
          class="btn btn-danger btn-sm"
          :disabled="!hasCompleted"
          @click="showClearConfirm = true"
        >清除已完成</button>
      </div>
    </div>

    <ul v-if="todos.length" class="todo-list">
      <li v-for="todo in todos" :key="todo.id" class="todo-item" :class="{ done: todo.done }">
        <label class="todo-label">
          <input
            type="checkbox"
            :checked="todo.done"
            @change="store.toggleTodo(planId, todo.id)"
          />
          <span class="todo-name">{{ todo.name }}</span>
        </label>
        <button
          type="button"
          class="todo-remove"
          title="删除"
          @click="store.removeTodo(planId, todo.id)"
        >×</button>
      </li>
    </ul>
    <div v-else class="empty">
      <div class="empty-icon">✓</div>
      <p>暂无待办事项</p>
    </div>

    <!-- 清除已完成二次确认 -->
    <Modal :show="showClearConfirm" title="清除已完成待办" @close="showClearConfirm = false">
      <p>
        确定要清除 <strong>{{ doneCount }}</strong> 项已完成的待办吗？未完成的待办不受影响，此操作不可撤销。
      </p>
      <template #footer>
        <button type="button" class="btn btn-ghost" @click="showClearConfirm = false">取消</button>
        <button type="button" class="btn btn-danger" @click="confirmClearCompleted">确认清除</button>
      </template>
    </Modal>
  </div>
</template>

<style scoped>
.todo-add {
  display: flex;
  gap: 8px;
}

.todo-bulk {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  flex-wrap: wrap;
}

.todo-count {
  font-size: 13px;
  color: var(--text-secondary);
}

.todo-bulk-actions {
  display: flex;
  gap: 8px;
}

.todo-list {
  list-style: none;
  margin-top: 4px;
}

.todo-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 9px 8px;
  border-bottom: 1px solid var(--border);
}

.todo-label {
  display: flex;
  align-items: center;
  gap: 10px;
  cursor: pointer;
  flex: 1;
}

.todo-label input {
  accent-color: var(--primary);
  width: 16px;
  height: 16px;
}

.todo-item.done .todo-name {
  text-decoration: line-through;
  color: var(--text-muted);
}

.todo-remove {
  border: none;
  background: transparent;
  color: var(--text-muted);
  font-size: 16px;
  opacity: 0;
}

.todo-item:hover .todo-remove {
  opacity: 1;
}

.todo-remove:hover {
  color: var(--danger);
}
</style>
