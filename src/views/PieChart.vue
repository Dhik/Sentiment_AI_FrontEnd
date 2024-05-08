<script setup>
import { onMounted, reactive, ref, watch } from 'vue';
import { ProductService } from '@/service/ProductService';
import { useLayout } from '@/layout/composables/layout';

const { layoutConfig } = useLayout();
let documentStyle = getComputedStyle(document.documentElement);
let textColor = documentStyle.getPropertyValue('--text-color');
let textColorSecondary = documentStyle.getPropertyValue('--text-color-secondary');
let surfaceBorder = documentStyle.getPropertyValue('--surface-border');

const products = ref(null);
const pieData = ref(null);

const pieOptions = ref(null);

const productService = new ProductService();

onMounted(() => {
    productService.getProductsSmall().then((data) => (products.value = data));
});
const setColorOptions = () => {
    documentStyle = getComputedStyle(document.documentElement);
    textColor = documentStyle.getPropertyValue('--text-color');
    textColorSecondary = documentStyle.getPropertyValue('--text-color-secondary');
    surfaceBorder = documentStyle.getPropertyValue('--surface-border');
};

const setChart = () => {
};

watch(
    layoutConfig.theme,
    () => {
        setColorOptions();
        setChart();
    },
    { immediate: true }
);
</script>


<template>
  <div class="flex justify-content-between mb-3">
    <div>
      <h2>Youtube</h2>
    </div>
    <div class="flex align-items-center justify-content-center bg-orange-100 border-round" style="width: 2.5rem; height: 2.5rem">
        <i class="pi pi-youtube text-orange-500 text-xl"></i>
    </div>
</div>
    <div class="grid">
      <div class="col-6">
        <div class="card">
          <h5>Searching Keyword</h5>
          <form @submit.prevent="fetchVideos" class="formgroup-inline">
            <div class="field">
              <label for="hashtag" class="p-sr-only">Enter Keyword</label>
              <InputText id="hashtag" type="text" v-model="hashtag" required />
            </div>
            <Button type="submit">Search</Button>
          </form>
        </div>
      </div>
      <div class="col-6">
        <div class="card">
          <h5>Filter</h5>
          <form @submit.prevent="addItem" autocomplete="off" class="formgroup-inline">
                <div class="field">
                    <InputText v-model="newItem" type="text" class="task-input"
                    />
                </div>
              <button class="submit">Add</button>
          </form>
          <ul class="task-list">
              <li
                class="task-list-item"
                v-for="(item, index) in items"
                :key="index"
                v-bind:class="{ completed: item.completed }"
              >
              <span class="task-text">{{ item.text }}</span>
              <button @click="deleteItem(index)" class="btn-delete">x</button>
              </li>
          </ul>
        </div>
      </div>
      <div class="col-6">
        <div class="card flex flex-column align-items-center" style="padding-left: 100px; padding-right: 100px;">
            <h5 class="text-left w-full">Sentiment Analysis from Youtube</h5>
            <canvas id="myPieChart" ></canvas>
        </div>
    </div>
    <div class="col-6">
      <div class="card">
        <h5>Word Cloud</h5>
        <svg ref="svg"></svg>
      </div>
    </div>
      <div class="col-12">
        <div class="card">
          <div v-if="videos.length === 0 && !loading">Enter a hashtag and click Fetch Videos.</div>
          <div v-else>
            <div v-for="(video, index) in videos" :key="index" class="tweet">
              <div class="tweet-content">
                <div class="tweet-header">
                  <img :src="video.profile_image" :alt="video.author" width="70" class="tweet-profile-pic">
                  <div class="tweet-info">
                    <span class="tweet-author">{{ video.author }}</span><br>
                    <span class="tweet-author">{{ video.text }}</span><br>
                    <span class="tweet-author">Like count: {{ video.likes}}</span><br>
                    <span class="tweet-price">{{ video.published_at }}</span>
                  </div>
                </div>
                <div class="tweet-description">
                  <!-- Add product description or any additional information here -->
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  

  <script>
  import * as d3 from "d3";
  import cloud from "d3-cloud";
  import Chart from 'chart.js/auto';
  export default {
    data() {
      return {
        hashtag: '',
        videos: [],
        loading: false,
        items: [],
      };
    },
    watch: {
        items: {
            handler(newItems) {
                if (newItems.length > 0) {
                    this.filterData();
                }
            },
            deep: true,
        }
    },
    methods: {
      renderChart(data) {
        let positiveCount = 0;
        let negativeCount = 0;
        data.forEach((item) => {
              if (item.sentiment === 'positif') {
                  positiveCount++;
              } else if (item.sentiment === 'negatif') {
                  negativeCount++;
              } else {
                  neutralCount++;
              }
          });
        const ctx = document.getElementById('myPieChart').getContext('2d');
        const myPieChart = new Chart(ctx, {
          type: 'pie',
          data: {
            labels: ['Negatif', 'Positif'],
            datasets: [{
              label: 'My Pie Chart',
              data: [negativeCount, positiveCount],
              backgroundColor: [
                'rgb(255, 99, 132)',
                'rgb(54, 162, 235)',
              ],
              hoverOffset: 4
            }]
          }
        });
      },
        filterData() {
            this.videos = this.videos.filter(video => 
                this.items.some(item => 
                    video.text.toLowerCase().includes(item.text.toLowerCase())
                )
            )
        },
  generateWordcloud() {
  // Define Indonesian stopwords
  const indonesianStopwords = [
    // Add your list of Indonesian stopwords here
    "dan", "atau", "sebagai", "di", "dari", "dalam", "ke", "pada", "oleh", "dengan", "yg", "yang", "ini", "itu", "gak", "bisa", "jadi", "ya", "harus", "nya", "lain", "aja", "lah", "lain", "ada", "cuma", "sy", "utk", "tau", "ga", "dg", "dpr", "untuk", "tdk", "org"
    // Add more stopwords as needed
  ];

  const wordsMap = {};
  this.videos.forEach(video => {
    const words = video.text.split(/\s+/); // Split text into words
    words.forEach(word => {
      const cleanedWord = word.toLowerCase().replace(/[^\w\s]/gi, ''); // Remove punctuation and convert to lowercase
      if (cleanedWord !== '' && !indonesianStopwords.includes(cleanedWord)) { // Check if word is not a stopword
        if (wordsMap[cleanedWord]) {
          wordsMap[cleanedWord]++;
        } else {
          wordsMap[cleanedWord] = 1;
        }
      }
    });
  });

  // Convert wordsMap to wc_data format
  const wc_data = Object.keys(wordsMap).map(word => ({
    text: word,
    size: wordsMap[word]
  }));

  const width = 500;
  const height = 250;

  const svg = d3.select(this.$refs.svg)
    .attr("width", width)
    .attr("height", height);

  const layout = cloud()
    .size([width, height])
    .words(wc_data)
    .padding(5)
    .rotate(() => (Math.random() < 0.5 ? 0 : 90))
    .font("Impact")
    .fontSize(d => d.size)
    .on("end", draw);

  layout.start();

  function draw(words) {
    svg.append("g")
      .attr("transform", `translate(${width / 2},${height / 2})`)
      .selectAll("text")
      .data(words)
      .enter().append("text")
      .style("font-size", d => `${d.size}px`)
      .style("font-family", "Impact")
      .style("fill", (d, i) => d3.schemeCategory10[i % 10])
      .attr("text-anchor", "middle")
      .attr("transform", d => `translate(${[d.x, d.y]})rotate(${d.rotate})`)
      .text(d => d.text);
  }
},
    addItem() {
      if (this.newItem.trim() !== "") {
        this.items.push({ text: this.newItem, completed: false });
        this.newItem = "";
        this.editingIndex = null;

        this.videos = this.videos.filter(video => 
            video.text && this.items.some(item => 
                video.text.toLowerCase().includes(item.text.toLowerCase())
            )
        )
        this.news = this.news.filter(news => 
            video.text && this.items.some(item => 
                news.title.toLowerCase().includes(item.text.toLowerCase())
            )
        )
      }
    },
    deleteItem(index) {
      this.items.splice(index, 1);
    },
      async fetchVideos() {
        if (!this.hashtag) {
          alert('Please enter a hashtag.');
          return;
        }
        this.loading = true;
        try {
          const response = await fetch('http://localhost:5000/api/youtube/comments', {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify({
              hashtag: this.hashtag
            })
          });
          const data = await response.json();
          this.videos = data;

          // console.log(typeof data[0].text);
          // console.log(data[0].text);

          // Generate word cloud after receiving video data
          this.generateWordcloud();
          this.renderChart(data);
  
          this.loading = false;
        } catch (error) {
          console.error('Error fetching videos:', error);
          this.loading = false;
        }
      },
      formatDate(dateString) {
        return new Date(dateString).toLocaleDateString();
      }
    },
  };
  </script>
  <style scoped>
  .formgroup-inline {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 10px;
  }
  
  .field {
      margin-right: 10px;
  }
  
  .task-input {
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
  }
  
  .submit {
      padding: 8px 16px;
      background-color: #007bff;
      color: #fff;
      border: none;
      border-radius: 4px;
      cursor: pointer;
  }
  
  .submit:hover {
      background-color: #0056b3;
  }
  
  .task-list {
      list-style-type: none;
      padding: 0;
  }
  
  .task-list-item {
      display: flex;
      align-items: center;
      margin-bottom: 5px;
  }
  
  .completed .task-text {
      text-decoration: line-through;
  }
  
  .btn-delete {
      background-color: #dc3545;
      color: #fff;
      border: none;
      border-radius: 50%;
      padding: 4px 8px;
      cursor: pointer;
  }
  
  .btn-delete:hover {
      background-color: #c82333;
  }
  
  .tweet {
      border: 1px solid #ccc;
      border-radius: 10px;
      padding: 10px;
      margin-bottom: 10px;
  }
  
  .tweet-content {
      flex: 1;
  }
  
  .tweet-header {
      display: flex;
      align-items: center;
  }
  
  .tweet-info {
      margin-left: 10px;
  }
  
  .tweet-actions a {
      color: #ffffff;
      margin-right: 10px;
      text-decoration: none;
  }
  
  .tweet-actions a:hover {
      text-decoration: underline;
  }
  
  .action-link {
      color: #657786;
      font-size: 14px;
      margin-right: 10px;
      text-decoration: none;
  }
  
  .action-link i {
      margin-right: 5px;
  }
  
  .action-link:hover {
      text-decoration: underline;
      color: #1da1f2;
  }
  .arc text {
    font: 10px sans-serif;
    text-anchor: middle;
  }
  canvas #myPieChart {
    aspect-ratio: none !important; /* or you can specify a custom aspect ratio if needed */
  }
  </style>

  