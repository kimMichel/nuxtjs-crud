<template>
  <div class="container mx-auto mt-8">
    <div v-if="error" class="text-center text-red-500">{{ error.message }}</div>

    <div class="flex justify-end mb-4">
      <button
        @click="openModal"
        class="bg-blue-500 text-white font-bold py-2 px-4 rounded"
      >
        Adicionar Novo Livro
      </button>
    </div>

    <div
      class="grid grid-cols-6 gap-4 bg-gray-200 text-gray-600 font-medium py-2 px-4"
    >
      <div>id</div>
      <div>Name</div>
      <div>Description</div>
      <div>Author Name</div>
      <div>Pages</div>
    </div>

    <div class="divide-y divide-gray-300">
      <div
        v-for="(item, index) in data"
        :key="item.id"
        class="grid grid-cols-6 gap-4 py-2 px-4 items-center"
      >
        <div>{{ index + 1 }}</div>
        <div>{{ item.name }}</div>
        <div>{{ item.description }}</div>
        <div>{{ item.authorName }}</div>
        <div>{{ item.pages }}</div>
        <div class="flex space-x-2">
          <button
            @click="openEditModal(item)"
            class="bg-yellow-500 text-white py-1 px-2 rounded"
          >
            Edit
          </button>
          <button
            @click="deleteItem(item.id)"
            class="bg-red-500 text-white py-1 px-2 rounded"
          >
            Delete
          </button>
        </div>
      </div>
    </div>

    <transition name="fade">
      <div
        v-if="isModalOpen"
        class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50"
      >
        <div class="bg-white rounded-lg p-6 w-96">
          <h2 class="text-lg font-bold mb-4">
            {{ isEditing ? "Editar Item" : "Adicionar Novo Item" }}
          </h2>
          <form @submit.prevent="isEditing ? updateBook() : addBook()">
            <div class="mb-4">
              <label class="block mb-1">Name</label>
              <input
                v-model="newItem.name"
                type="text"
                class="border border-gray-300 p-2 w-full rounded"
                required
              />
            </div>
            <div class="mb-4">
              <label class="block mb-1">Description</label>
              <textarea
                v-model="newItem.description"
                class="border border-gray-300 p-2 w-full rounded"
                required
              ></textarea>
            </div>
            <div class="mb-4">
              <label class="block mb-1">Author Name</label>
              <input
                v-model="newItem.authorName"
                type="text"
                class="border border-gray-300 p-2 w-full rounded"
                required
              />
            </div>
            <div class="mb-4">
              <label class="block mb-1">Pages</label>
              <input
                v-model="newItem.pages"
                type="number"
                class="border border-gray-300 p-2 w-full rounded"
              />
            </div>
            <div class="flex justify-end">
              <button
                type="button"
                @click="closeModal"
                class="text-gray-500 mr-2"
              >
                Cancelar
              </button>
              <button
                type="submit"
                class="bg-blue-500 text-white py-2 px-4 rounded"
              >
                {{ isEditing ? "Editar" : "Adicionar" }}
              </button>
            </div>
          </form>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref } from "vue";

const isModalOpen = ref(false);
const isEditing = ref(false);
const newItem = ref({
  name: "",
  description: "",
  authorName: "",
  pages: null,
});
const currentItemId = ref(null);

const openModal = () => {
  isModalOpen.value = true;
  isEditing.value = false;
};

const closeModal = () => {
  isModalOpen.value = false;

  resetForm();
};

const resetForm = () => {
  newItem.value = {
    name: "",
    description: "",
    authorName: "",
    pages: null,
  };
  currentItemId.value = null;
};

const openEditModal = (item) => {
  isEditing.value = true;
  newItem.value = { ...item };
  currentItemId.value = item.id;
  isModalOpen.value = true;
};

const addBook = async () => {
  try {
    const response = await fetch("http://localhost:3333/books", {
      method: "POST",
      headers: {
        "Content-type": "application/json",
      },
      body: JSON.stringify(newItem.value),
    });

    if (!response.ok) {
      throw new Error("Error when add new book.");
    }

    fetchBooksAfterChange();
    closeModal();
  } catch (error) {
    console.error("Error when try to add new book:", error);
  }
};

const updateBook = async () => {
  try {
    const response = await fetch(
      `http://localhost:3333/books/${currentItemId.value}`,
      {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(newItem.value),
      }
    );

    if (!response.ok) {
      throw new Error("Error when edit a book info.");
    }

    await fetchBooksAfterChange();
    closeModal();
  } catch (error) {
    console.error("Error when try edit a book info:", error);
  }
};

const deleteItem = async (id) => {
  if (confirm("Tem certeza que deseja deletar este item?")) {
    try {
      const response = await fetch(`http://localhost:3333/books/${id}`, {
        method: "DELETE",
      });

      if (!response.ok) {
        throw new Error("Error when delete item.");
      }

      await fetchBooksAfterChange();
    } catch (error) {
      console.error("Error when try delete item:", error);
    }
  }
};

const fetchBooks = async () => {
  const { data, error } = await useFetch("http://localhost:3333/books");
  return { data, error };
};

const fetchBooksAfterChange = async () => {
  const fetchResult = await fetchBooks();
  data.value = fetchResult.booksData.value;
};

const { data, error } = await fetchBooks();

if (error.value) {
  console.error("Error when try to fetch data:", error.value);
}
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
