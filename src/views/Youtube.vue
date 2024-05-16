<script setup>
import { onMounted, reactive, ref, watch } from 'vue';
import { ProductService } from '@/service/ProductService';
import { useLayout } from '@/layout/composables/layout';


</script>


<template>
  <div class="flex justify-content-between mb-3">
    <div>
      <h2>Youtube</h2>
    </div>
    <div class="flex align-items-center justify-content-center bg-orange-100 border-round"
      style="width: 2.5rem; height: 2.5rem">
      <i class="pi pi-youtube text-orange-500 text-xl"></i>
    </div>
  </div>
  <div class="grid">
    <div class="col-6">
      <div class="card">
        <div class="grid">
          <div class="col-6">
            <h5>Start Date</h5>
            <div class="field">
              <label for="start-date" class="p-sr-only">Start Date</label>
              <Calendar :showIcon="true" id="start-date" v-model="startDate"></Calendar>
            </div>
          </div>
          <div class="col-6">
            <h5>End Date</h5>
            <div class="field">
              <label for="end-date" class="p-sr-only">End Date</label>
              <Calendar :showIcon="true" id="end-date" v-model="endDate"></Calendar>
            </div>
          </div>
        </div>
        <form @submit.prevent="fetchVideos" class="formgroup-inline">
          <Button type="submit">Search</Button>
        </form>
      </div>
    </div>
    <div class="col-6">
      <div class="card">
        <h5>Filter</h5>
        <form @submit.prevent="addItem" autocomplete="off" class="formgroup-inline">
          <div class="field">
            <InputText v-model="newItem" type="text" class="task-input" />
          </div>
          <Button type="submit">Add</Button>
        </form>
        <ul class="task-list">
          <li class="btn task-list-item" v-for="(item, index) in items" :key="index"
            v-bind:class="{ completed: item.completed }">
            <span class="task-text">{{ item.text }}</span>
            <button @click="deleteItem(index)" class="btn-delete">x</button>
          </li>
        </ul>
        <h5>Perspektif</h5>
        <form @submit.prevent="fetchVideos" class="formgroup-inline">
          <div class="field">
            <label for="hashtag" class="p-sr-only">Enter Keyword</label>
            <InputText id="perspektif" type="text" v-model="perspektif" required />
          </div>
          <Button type="submit">Search</Button>
        </form>

      </div>
    </div>
    <div class="col-3">
      <div class="card">
        <div class="flex justify-content-between mb-3" v-if="!loading && videos.length > 0">
          <div>
            <span class="block text-500 font-medium mb-3">{{ hashtag }}</span>
            <div class="text-900 font-medium text-xl">{{ videos.length }} Chat</div>
          </div>
          <div class="flex align-items-center justify-content-center bg-orange-100 border-round"
            style="width: 2.5rem; height: 2.5rem">
            <i class="pi pi-youtube text-orange-500 text-xl"></i>
          </div>
        </div>
        <span class="text-green-500 font-medium">{{ countSentiment('positif') }}&nbsp;</span>
        <span class="text-500">Positif &nbsp;&nbsp;</span>
        <span class="text-red-500 font-medium">{{ countSentiment('negatif') }}&nbsp;</span>
        <span class="text-500">Negatif</span>
      </div>
      <div class="card">
        <h5>Trending Topics</h5>
        <ul class="list-none p-0 m-0">
          <li class="flex flex-column md:flex-row md:align-items-center md:justify-content-between mb-4">
            <div>
              <span class="text-900 font-medium mr-2 mb-1 md:mb-0">1. {{ topic["1"] }}</span>
            </div>
            <div class="mt-2 md:mt-0 flex align-items-center">
              <!-- <span class="text-orange-500 ml-3 font-medium">50</span> -->
            </div>
          </li>
          <li class="flex flex-column md:flex-row md:align-items-center md:justify-content-between mb-4">
            <div>
              <span class="text-900 font-medium mr-2 mb-1 md:mb-0">2. {{ topic["2"] }}</span>
            </div>
            <div class="mt-2 md:mt-0 ml-0 md:ml-8 flex align-items-center">
              <!-- <span class="text-orange-500 ml-3 font-medium">16</span> -->
            </div>
          </li>
          <li class="flex flex-column md:flex-row md:align-items-center md:justify-content-between mb-4">
            <div>
              <span class="text-900 font-medium mr-2 mb-1 md:mb-0">3. {{ topic["3"] }}</span>
            </div>
            <div class="mt-2 md:mt-0 ml-0 md:ml-8 flex align-items-center">
              <!-- <span class="text-orange-500 ml-3 font-medium">67</span> -->
            </div>
          </li>
        </ul>
      </div>
    </div>
    <div class="col-3">
      <!-- <div class="col-12">
          <div class="card flex flex-column align-items-center">
            <h5 class="text-left w-full">Sentiment Analysis from Youtube</h5>
          </div>
        </div> -->

      <div class="card flex flex-column align-items-center">
        <h5 class="text-left w-full">Sentiment Analysis from Youtube</h5>
        <canvas id="myPieChart"></canvas>
      </div>
    </div>
    <div class="col-6">
      <div class="card">
        <h5>Word Cloud</h5>
        <svg ref="svg"></svg>
      </div>
    </div>
    <div class="col-6">
      <div class="card">
        <h5>Sentimen Positif</h5>
        <div v-if="filteredVideos.length === 0 && !loading">Enter a hashtag and click Fetch Videos.</div>
        <div v-else class="tweet-container">
          <div v-for="(video, index) in filteredVideos" :key="index" class="tweet">
            <div class="tweet-content">
              <div class="tweet-header">
                <img :src="video.profile_image" :alt="video.author" width="70" class="tweet-profile-pic">
                <div class="tweet-info">
                  <span class="tweet-author">{{ video.author }}</span><br>
                  <span class="tweet-author">{{ video.text }}</span><br>
                  <span class="tweet-author">Like count: {{ video.likes }}</span><br>
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


    <div class="col-6">
      <div class="card">
        <h5>Sentimen Negatif</h5>
        <div v-if="videos.length === 0 && !loading">Enter a hashtag and click Fetch Videos.</div>
        <div v-else class="tweet-container">
          <div v-for="(video, index) in filteredVideos2" :key="index" class="tweet">
            <div class="tweet-content">
              <div class="tweet-header">
                <img :src="video.profile_image" :alt="video.author" width="70" class="tweet-profile-pic">
                <div class="tweet-info">
                  <span class="tweet-author">{{ video.author }}</span><br>
                  <span class="tweet-author">{{ video.text }}</span><br>
                  <span class="tweet-author">Like count: {{ video.likes }}</span><br>
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
      hashtag: 'ruupenyiaran',
      perspektif: '',
      videos: [],
      ori_videos: [],
      loading: false,
      items: [],
      topic: {
        "1": "DPD tidak",
        "2": "korupsi DPR",
        "3": "perampasan aset"
      },
      startDate: '',
      endDate: '',
    };
  },
  computed: {
    filteredVideos() {
      return this.videos.filter(video => video.sentiment === "positif");
    },
    filteredVideos2() {
      return this.videos.filter(video => video.sentiment === "negatif");
    }
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
    countSentiment(sentiment) {
      return this.videos.filter(video => video.sentiment === sentiment).length;
    },
    renderChart(data) {
      let positiveCount = 0;
      let negativeCount = 0;
      let neutralCount = 0;
      data.forEach((item) => {
        if (item.sentiment === 'positif') {
          positiveCount++;
        } else if (item.sentiment === 'negatif') {
          negativeCount++;
        } else {
          neutralCount++;
        }
      });

      const canvas = document.getElementById('myPieChart');
      const ctx = canvas.getContext('2d');

      // Check if a chart instance already exists
      if (canvas.chart) {
        // Destroy the existing chart
        canvas.chart.destroy();
      }

      // Render the new chart
      canvas.chart = new Chart(ctx, {
        type: 'pie',
        data: {
          labels: ['Negatif', 'Positif'],
          datasets: [{
            label: 'Jumlah komentar',
            data: [negativeCount, positiveCount],
            backgroundColor: [
              'rgb(255, 99, 132)',
              'rgb(54, 162, 235)',
            ],
            hoverOffset: 4
          }]
        },
        options: {
          plugins: {
            datalabels: {
              formatter: (value, ctx) => {
                let sum = 0;
                let dataArr = ctx.chart.data.datasets[0].data;
                dataArr.map(data => {
                  sum += data;
                });
                let percentage = (value * 100 / sum).toFixed(2) + "%";
                return percentage;
              },
              color: '#fff'
            }
          }
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
        "dan", "atau", "sebagai", "di", "dari", "dalam", "ke", "pada", "oleh", "dengan", "yg", "yang", "ini", "itu", "gak", "bisa", "jadi", "ya", "harus", "nya", "lain", "aja", "lah", "lain", "ada", "cuma", "sy", "utk", "tau", "ga", "dg", "dpr", "untuk", "tdk", "org",
        "kok", "apa", "tapi", "pak", "tidak", "para", "kalian", "saya", "kan", "klo", "jd", "pd", "sj", "sja", "lg", "orng", "aku", "toh", "bg", "ia", "gue", "a", "hai", "sm", "aq", "kl", "jg", "bgt", "ni", "tuk", "dia", "g", "bru", "ji", "sg", "l", "r", "hi", "kita", "orang", "ud",
        "th", "pp", "lagi", "ntar", "a", "b", "c", "d", "e", "f", "h", "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s", "t", "v", "w", "x", "y", "z", "ky", "td", "pj", "kha", "tp", "dll", "gk", "klw",
        "sih", "ttp", "as", "uhuy", "ae", "lha", "tpi", "dn", "no", "eh", "gw", "so", ""
        // Add more stopwords as needed
      ];

      const wordsMap = {};
      this.videos.forEach(video => {
        const words = video.text.split(/\s+/); // Split text into words
        words.forEach(word => {
          const cleanedWord = word.toLowerCase().replace(/[^\w\s]|[\d]/gi, ''); // Remove punctuation and convert to lowercase
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

      const width = 530;
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
        .fontSize(d => d.size * 5) // Increase font size by multiplying by a factor (e.g., 1.5)
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

    filterArrays() {
      this.videos = this.ori_videos.filter(video =>
        this.items.every(item =>
          video.text.toLowerCase().includes(item.text.toLowerCase())
        )
      );

      // this.news = this.news.filter(news => 
      //   news.title && this.items.some(item => 
      //     news.title.toLowerCase().includes(item.text.toLowerCase())
      //   )
      // );
    },
    removeWordCloud() {
      const svg = this.$refs.svg;
      while (svg.firstChild) {
        svg.removeChild(svg.firstChild);
      }
    },
    addItem() {
      if (this.newItem.trim() !== "") {
        this.items.push({ text: this.newItem, completed: false });
        this.newItem = "";
        this.editingIndex = null;
        this.videos = this.ori_videos;
        this.filterArrays();
        this.renderChart(this.videos);
        this.removeWordCloud();
        this.generateWordcloud();
      }
    },
    deleteItem(index) {
      this.items.splice(index, 1);
      this.videos = this.ori_videos;
      this.filterArrays();
      this.renderChart(this.videos);
      this.removeWordCloud();
      this.generateWordcloud();
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
            hashtag: this.hashtag,
            perspektif: ''
          })
        });
        const data = await response.json();
        let filteredComments = data.comments;

        if (this.startDate) {
          const startDate = new Date(this.startDate);
          filteredComments = filteredComments.filter(comment => new Date(comment.published_at) >= startDate);
        }
        if (this.endDate) {
          const endDate = new Date(this.endDate);
          filteredComments = filteredComments.filter(comment => new Date(comment.published_at) <= endDate);
        }

        this.videos = filteredComments;
        this.ori_videos = filteredComments;
        // this.topic = data.topics;

        // console.log(typeof data[0].text);
        // console.log(data[0].text);

        // Generate word cloud after receiving video data
        this.generateWordcloud();
        this.renderChart(data.comments);

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
  // mounted() {
  //   this.fetchVideos();
  // }
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
  display: flex;
  /* Add display flex */
  flex-wrap: wrap;
  /* Allow items to wrap to the next line if needed */
}

.task-list-item {
  background-color: #f0f0f0;
  /* Background color */
  color: #333;
  /* Text color */
  border: 1px solid #ccc;
  /* Border */
  border-radius: 5px;
  /* Rounded corners */
  padding: 8px 12px;
  /* Padding for spacing */
  cursor: pointer;
  /* Cursor on hover */
  transition: background-color 0.3s, color 0.3s, border-color 0.3s;
  /* Smooth transitions */
  display: inline-block;
  /* Display as inline block */
  margin-right: 10px;
  /* Add margin to create space between items */
}

.completed .task-text {
  text-decoration: line-through;
}

.btn-delete {
  color: black;
  border: none;
  border-radius: 50%;
  padding: 4px 8px;
  margin-left: 6px;
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
  aspect-ratio: none !important;
  /* or you can specify a custom aspect ratio if needed */
}

.tweet-container {
  max-height: 400px;
  /* Adjust the height as needed */
  overflow-y: auto;
}
</style>