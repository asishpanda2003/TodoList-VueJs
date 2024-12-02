<script setup>
import { ref, onMounted, computed, watch } from "vue";

const todos = ref([]);
const name = ref("");
const input_content = ref("");
const inpput_category = ref(null);
const filterCategory = ref("All"); // For dropdown filter

const todo_asc = computed(() =>
  todos.value.sort((a, b) => b.createdAt - a.createdAt)
);

const filteredTodos = computed(() => {
  if (filterCategory.value === "All") {
    return todo_asc.value;
  }
  return todo_asc.value.filter(
    (todo) => todo.category === filterCategory.value
  );
});

const addTodo = () => {
  if (input_content.value.trim() === "" || inpput_category.value === null) {
    alert("Please select a category and enter a TODO before adding.");
    return;
  }
  todos.value.push({
    content: input_content.value,
    category: inpput_category.value,
    done: false,
    createdAt: new Date().getTime(),
  });
  input_content.value = "";
  inpput_category.value = null; // Reset category selection
};

const getCategoryColor = (category) => {
  if (category === "Business") return "#17a2b8"; // Info color
  if (category === "Personal") return "#ffc107"; // Warning color
  return "#6c757d"; // Default gray color
};

const removeTodo = (todo) => {
  todos.value = todos.value.filter((t) => t !== todo);
};

watch(
  todos,
  (newVal) => {
    localStorage.setItem("todos", JSON.stringify(newVal));
  },
  { deep: true }
);

watch(name, (newVal) => {
  localStorage.setItem("name", newVal);
});

onMounted(() => {
  name.value = localStorage.getItem("name") || "";
  todos.value = JSON.parse(localStorage.getItem("todos")) || [];
});
</script>

<template>
  <main class="container py-4">
    <!-- Greeting Section -->
    <section class="text-center mb-4">
      <h2 class="mb-3">
        What's up,
        <input
          type="text"
          class="form-control d-inline w-auto text-bold h1 font-weight-900 border border-primary shadow-p p-2 interactive-input"
          placeholder="Name here"
          v-model="name"
        />
      </h2>
    </section>

    <!-- Create To-Do Section -->
    <section class="bg-white p-4 rounded shadow mb-4">
      <h3 class="text-primary mb-3">CREATE A TODO</h3>
      <form @submit.prevent="addTodo">
        <div class="mb-3">
          <h4 class="text-secondary">What's on your Todo list</h4>
          <input
            type="text"
            class="form-control"
            placeholder="e.g., Make some todo"
            v-model="input_content"
          />
        </div>
        <div class="mb-3">
          <h4 class="text-secondary">Pick a category</h4>
          <div class="d-flex gap-3">
            <label
              class="form-check-label p-3 border rounded d-flex flex-column align-items-center shadow w-50"
              :class="{
                'bg-info bg-gradient text-white':
                  inpput_category === 'Business',
              }"
            >
              <input
                type="radio"
                class="form-check-input"
                name="category"
                value="Business"
                v-model="inpput_category"
              />
              <div class="mt-2">Business</div>
            </label>
            <label
              class="form-check-label p-3 border rounded d-flex flex-column align-items-center shadow w-50"
              :class="{
                'bg-warning bg-gradient text-white':
                  inpput_category === 'Personal',
              }"
            >
              <input
                type="radio"
                class="form-check-input"
                name="category"
                value="Personal"
                v-model="inpput_category"
              />
              <div class="mt-2">Personal</div>
            </label>
          </div>
        </div>
        <button type="submit" class="btn btn-primary w-100">Add Todo</button>
      </form>
    </section>

    <!-- To-Do List Section -->
    <section class="bg-white p-4 rounded shadow">
      <div class="d-flex justify-content-between align-items-center mb-3">
        <h3 class="text-primary">TODO LIST</h3>
        <select class="form-select w-auto" v-model="filterCategory">
          <option value="All">All</option>
          <option value="Business">Business</option>
          <option value="Personal">Personal</option>
        </select>
      </div>
      <div>
        <div
          v-for="todo in filteredTodos"
          :key="todo.createdAt"
          class="d-flex align-items-center mb-3 p-3 rounded shadow"
          :class="{ 'bg-light text-muted': todo.done }"
        >
          <!-- Checkbox -->
          <label class="me-3">
            <input
              type="checkbox"
              class="form-check-input"
              v-model="todo.done"
            />
          </label>

          <!-- Category Color Indicator -->
          <div
            class="rounded-circle me-3"
            :style="{
              width: '20px',
              height: '20px',
              backgroundColor: getCategoryColor(todo.category),
            }"
          ></div>

          <!-- Todo Content -->
          <div class="flex-grow-1">
            <div class="d-flex flex-column">
              <template v-if="todo.done">
                <del>{{ todo.content }}</del>
              </template>
              <template v-else>
                <input
                  type="text"
                  class="form-control border-0"
                  v-model="todo.content"
                />
              </template>
            </div>
          </div>

          <!-- Delete Button -->
          <button class="btn btn-danger btn-sm ms-3" @click="removeTodo(todo)">
            Delete
          </button>
        </div>
      </div>
    </section>
  </main>
</template>
