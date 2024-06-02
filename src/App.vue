<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input" /><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button save-button">{{ form.id ? 'Update' : 'Save' }}</button>
      <p v-if="errorMessage" class="error">{{ errorMessage }}</p>
    </form>

    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h3>{{ article.title }}</h3>
        <p>{{ article.content }}</p>
        <button @click="edit(article)" class="button edit-button">Edit</button>
        <button @click="remove(article.id)" class="button delete-button">Delete</button>
      </li>
    </ul>

    <button @click="load" class="button load-button">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue'
import axios from 'axios'

export default {
  setup() {
    const form = reactive({
      id: '',
      title: '',
      content: ''
    })

    const articles = ref([])
    const errorMessage = ref('')

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles')
        articles.value = response.data
        console.log('Articles loaded:', articles.value)
      } catch (error) {
        console.error('Error loading articles:', error)
      }
    }

    function generateRandomId() {
      return Math.random().toString(36).substr(2, 9)
    }

    async function save() {
      if (!form.title || !form.content) {
        errorMessage.value = 'Title and content must not be empty.'
        return
      }
      errorMessage.value = ''
      try {
        let url = 'http://localhost:3000/articles'
        let method = 'post'

        if (form.id) {
          url = `http://localhost:3000/articles/${form.id}`
          method = 'put'
        } else {
          form.id = generateRandomId()
        }

        const response = await axios[method](url, form)

        if (form.id) {
          const index = articles.value.findIndex((article) => article.id == form.id)
          if (index !== -1) {
            articles.value[index] = response.data
          } else {
            articles.value.push(response.data)
          }
        }

        form.id = ''
        form.title = ''
        form.content = ''
        console.log('Article saved:', response.data)
      } catch (error) {
        console.error('Error saving article:', error)
      }
    }

    async function remove(id) {
      try {
        const response = await axios.delete(`http://localhost:3000/articles/${id}`)
        console.log('Delete response:', response)
        if (response.status === 200) {
          articles.value = articles.value.filter((article) => article.id !== id)
          console.log('Article deleted:', id)
        } else {
          console.error('Failed to delete article:', response)
        }
      } catch (error) {
        console.error('Error deleting article:', error)
      }
    }

    function edit(article) {
      form.id = article.id
      form.title = article.title
      form.content = article.content
      console.log('Editing article:', article)
    }

    onMounted(() => {
      load()
    })

    return {
      form,
      articles,
      save,
      edit,
      remove,
      errorMessage
    }
  }
}
</script>

<style>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.form {
  margin-bottom: 20px;
}

.input,
.textarea {
  width: 100%;
  padding: 10px;
  margin: 10px 0;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.button {
  padding: 10px 15px;
  margin: 5px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.save-button {
  background-color: #4caf50;
  color: white;
}

.edit-button {
  background-color: #2196f3;
  color: white;
}

.delete-button {
  background-color: #f44336;
  color: white;
}

.load-button {
  background-color: #ff9800;
  color: white;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  border-bottom: 1px solid #ddd;
  padding: 10px 0;
}

.article-item h3 {
  margin: 0;
}

.article-item p {
  margin: 5px 0 10px;
}

.error {
  color: red;
}
</style>
