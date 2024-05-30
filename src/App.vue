<template>
  <div class="container">
    <!-- Form -->
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input-field"/><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea-field"></textarea><br />
      <button type="submit" class="save-button">Save</button>
    </form>

    <!-- List of Articles -->
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <button @click="edit(article)" class="edit-button">Edit</button>
        <button @click="deleteArticle(article.id)" class="delete-button">Delete</button>
      </li>
    </ul>

    <!-- Load Button -->
    <button @click="load" class="load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: '',
      title: '',
      content: '',
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, { title: form.title, content: form.content });

        // Assuming successful save, update UI and reset form
        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          articles.value.push(response.data);
        }

        // Reset form
        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function deleteArticle(id) {
      try {
        // Hapus artikel dari array lokal terlebih dahulu
        articles.value = articles.value.filter((article) => article.id !== id);
        // Kemudian kirim permintaan hapus ke server
        await axios.delete(`http://localhost:3000/articles/${id}`);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, deleteArticle, load };
  },
};
</script>

<style>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 30px;
  font-family: Arial, sans-serif;
  background-color: rgba(255, 255, 255, 0.9);
  border-radius: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.form {
  margin-bottom: 30px;
}

.input-field, .textarea-field {
  width: 100%;
  padding: 15px;
  margin-bottom: 15px;
  border: 1px solid #ccc;
  border-radius: 8px;
  transition: border-color 0.3s, box-shadow 0.3s;
}

.input-field:focus, .textarea-field:focus {
  border-color: #1e88e5;
  box-shadow: 0 0 8px rgba(30, 136, 229, 0.2);
  outline: none;
}

.save-button, .load-button, .edit-button, .delete-button {
  padding: 12px 18px;
  margin-right: 10px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s, transform 0.2s, box-shadow 0.3s;
}

.save-button {
  background-color: #4CAF50;
  color: white;
}

.save-button:hover {
  background-color: #45a049;
  transform: translateY(-3px);
  box-shadow: 0 4px 10px rgba(76, 175, 80, 0.3);
}

.load-button {
  background-color: #1e88e5;
  color: white;
}

.load-button:hover {
  background-color: #1565c0;
  transform: translateY(-3px);
  box-shadow: 0 4px 10px rgba(30, 136, 229, 0.3);
}

.edit-button {
  background-color: #ffa000;
  color: white;
}

.edit-button:hover {
  background-color: #fb8c00;
  transform: translateY(-3px);
  box-shadow: 0 4px 10px rgba(255, 160, 0, 0.3);
}

.delete-button {
  background-color: #f44336;
  color: white;
}

.delete-button:hover {
  background-color: #e53935;
  transform: translateY(-3px);
  box-shadow: 0 4px 10px rgba(244, 67, 54, 0.3);
}

.article-list {
  list-style: none;
  padding: 0;
}

.article-item {
  background-color: rgba(255, 255, 255, 0.8);
  padding: 25px;
  margin-bottom: 20px;
  border: 1px solid #ddd;
  border-radius: 10px;
  transition: transform 0.2s, box-shadow 0.3s;
}

.article-item:hover {
  transform: scale(1.03);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.article-item h3 {
  margin-top: 0;
  color: #333;
}

.article-item p {
  margin: 15px 0;
  color: #666;
}

/* General Body Styling */
body {
  font-family: 'Arial', sans-serif;
  color: #333;
  margin: 0;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100vh;
  background: linear-gradient(135deg, #a1c4fd, #c2e9fb, #d4fc79, #96e6a1);
  background-size: 400% 400%;
  animation: gradient 15s ease infinite;
}

@keyframes gradient {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

</style>
