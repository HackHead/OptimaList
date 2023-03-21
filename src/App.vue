<template>
  <div class="container mx-auto max-w-4xl mt-40 p-4 bg-white">
    <h1 class="text-2xl font-bold mb-8 text-center">Todo App</h1>

    <div class="mb-4">
      <input
        type="text"
        class="w-full rounded-lg p-3 text-sm border-gray-100 border-2"
        placeholder="Здійснити пошук"
        v-model="searchQuery"
      />
    </div>

    <form @submit.prevent="addTodo">
      <div class="mb-4">
        <div class="flex">
          <input
            type="text"
            class="w-full rounded-lg p-3 text-sm border-gray-100 border-2"
            placeholder="Заголовок завдання"
            v-model="newTodo.text"
            @input="saveDraft"
          />
          <input
            type="text"
            class="w-full rounded-lg p-3 text-sm border-gray-100 border-2"
            placeholder="Введіть теги, розділені комою"
            v-model="newTodo.tags"
            @input="saveDraft"
          />
        </div>
        <div class="text-center mt-4">
          <button
            class="inline-block w-full rounded-lg bg-black px-5 py-3 font-medium text-white sm:w-auto hover:text-black hover:border-black border-2 hover:bg-white"
            type="submit"
          >
            Додати завдання
          </button>
        </div>
      </div>
    </form>

    <article
      class="rounded-xl bg-white p-4 ring ring-indigo-50 sm:p-6 lg:p-8 my-4"
      v-for="(todo, index) in filteredTodos"
      :key="index"
    >
      <div class="flex items-start sm:gap-8">
        <div class="w-full">
          <div class="flex justify-between">
            <div>
              <label
                :for="index + `AcceptConditions`"
                class="relative h-8 w-14 cursor-pointer block"
                ><input
                  type="checkbox"
                  :id="index + `AcceptConditions`"
                  v-model="todo.completed"
                  class="peer sr-only [&amp;:checked_+_span_svg[data-unchecked-icon]]:hidden [&amp;:checked_+_span_svg[data-checked-icon]]:block" /><span
                  class="absolute inset-0 z-10 m-1 inline-flex h-6 w-6 items-center justify-center rounded-full bg-white text-gray-400 transition peer-checked:translate-x-6 peer-checked:text-green-600"
                  :checked="todo.completed"
                  ><svg
                    data-unchecked-icon="true"
                    xmlns="http://www.w3.org/2000/svg"
                    class="h-4 w-4"
                    viewBox="0 0 20 20"
                    fill="currentColor"
                  >
                    <path
                      fill-rule="evenodd"
                      d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z"
                      clip-rule="evenodd"
                    ></path></svg
                  ><svg
                    data-checked-icon="true"
                    xmlns="http://www.w3.org/2000/svg"
                    class="hidden h-4 w-4"
                    viewBox="0 0 20 20"
                    fill="currentColor"
                  >
                    <path
                      fill-rule="evenodd"
                      d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z"
                      clip-rule="evenodd"
                    ></path></svg></span
                ><span
                  class="absolute inset-0 rounded-full bg-gray-300 transition peer-checked:bg-green-500"
                ></span
              ></label>
            </div>
            <div>
              <button
                type="button"
                @click="deleteTodo(index)"
                class="inline-block w-full rounded-lg bg-black px-5 py-3 font-medium text-white sm:w-auto hover:text-black hover:border-black border-2 hover:bg-white"
              >
                Видалити
              </button>
            </div>
          </div>
          <h3
            class="mt-4 text-lg font-medium sm:text-xl"
            :class="{ 'line-through': todo.completed }"
          >
            <a href=""> {{ todo.text }} </a>
          </h3>
          <p class="mt-1 text-sm text-gray-700">
            <span
              v-for="(tag, index) in todo.tags"
              :key="index"
              class="whitespace-nowrap rounded-full bg-purple-100 px-2.5 py-0.5 text-sm text-purple-700"
            >
              {{ tag }}
            </span>
          </p>
        </div>
      </div>
    </article>
  </div>
</template>

<script>
import { ref, computed, onMounted, watch } from "vue";

export default {
  name: "TodoApp",
  setup() {
    const newTodo = ref({
      text: "",
      tags: "",
    });

    const todos = ref(JSON.parse(localStorage.getItem("todos")) || []);
    const searchQuery = ref(localStorage.getItem("searchQuery") || "");

    watch(searchQuery, (newValue) => {
      localStorage.setItem("searchQuery", newValue);
    });

    const filteredTodos = computed(() => {
      return todos.value.filter((todo) => {
        return (
          todo.text.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
          todo.tags.join(", ").toLowerCase().includes(searchQuery.value.toLowerCase())
        );
      });
    });

    const addTodo = () => {
      if (newTodo.value.text.trim() !== "") {
        todos.value.push({
          text: newTodo.value.text,
          tags: newTodo.value.tags
            .split(",")
            .map((tag) => tag.trim())
            .filter((tag) => tag.length),
          completed: false,
        });

        // Clear form
        newTodo.value.text = "";
        newTodo.value.tags = "";

        localStorage.setItem("todos", JSON.stringify(todos.value));
        localStorage.setItem("newTodoText", "");
        localStorage.setItem("newTodoTags", "");
      } else {
        alert("Довжина заголовку повинна бути довша від нуля")
      }
    };

    const deleteTodo = (index) => {
      todos.value.splice(index, 1);
      localStorage.setItem("todos", JSON.stringify(todos.value));
    };

    const saveDraft = () => {
      localStorage.setItem("newTodoText", newTodo.value.text);
      localStorage.setItem("newTodoTags", newTodo.value.tags);
    };

    const loadDraft = () => {
      const newTodoText = localStorage.getItem("newTodoText");
      const newTodoTags = localStorage.getItem("newTodoTags");
      if (newTodoText !== null) {
        newTodo.value.text = newTodoText;
      }
      if (newTodoTags !== null) {
        newTodo.value.tags = newTodoTags;
      }
    };

    onMounted(() => {
      loadDraft();
    });

    return {
      newTodo,
      todos,
      searchQuery,
      filteredTodos,
      addTodo,
      deleteTodo,
      saveDraft,
    };
  },
};
</script>

<style>
  .tags {
    display: flex;
    flex-wrap: wrap;
  }

  .tags span {
    background-color: #edf2f7;
    color: #4a5568;
    border-radius: 0.25rem;
    padding: 0.25rem 0.5rem;
    font-size: 0.75rem;
    margin-right: 0.5rem;
    margin-bottom: 0.5rem;
  }
</style>
