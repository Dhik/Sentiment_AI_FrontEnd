<template>
    <div>
      <h1>News Search</h1>
      <form @submit.prevent="fetchNews">
        <label for="query">Enter Query:</label>
        <input type="text" id="query" v-model="query" required>
        <button type="submit">Search News</button>
      </form>
      <div v-if="loading">Loading news...</div>
      <div v-else>
        <div v-for="(article, index) in articles" :key="index">
          <h2>{{ article.title }}</h2>
          <p>Source: {{ article.source_name }}</p>
          <p>Date: {{ article.date }}</p>
          <p><a :href="article.link" target="_blank">Read more</a></p>
          <img :src="article.source_icon" :alt="article.title" width="100">
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue';
  
  const query = ref('');
  const articles = ref([]);
  const loading = ref(false);
  
  const fetchNews = () => {
    if (!query.value) {
      alert('Please enter a query.');
      return;
    }
    loading.value = true;
    fetch('http://localhost:5000/news', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        query: query.value
      })
    })
    .then(response => response.json())
    .then(data => {
      articles.value = data;
      loading.value = false;
    })
    .catch(error => {
      console.error('Error fetching news:', error);
      loading.value = false;
    });
  };
  </script>
  