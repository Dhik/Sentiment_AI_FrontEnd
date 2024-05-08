<template>
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
      <!-- <div class="col-6">
        <div class="card">
          <div v-if="videos.length === 0 && !loading">No videos found for the keyword "{{ videoKeyword }}".</div>
          <div v-else>
            <div v-for="(video, index) in filteredVideos" :key="index" class="tweet">
                <div class="tweet-content">
                    <div class="tweet-header">
                      <img :src="video.thumbnails" :alt="video.title" width="70" class="tweet-profile-pic">
                      <div class="tweet-info">
                        <span class="tweet-author">{{ video.title }}</span><br>
                        <span class="tweet-author">{{ video.channelTitle }}</span><br>
                        <span class="tweet-author">{{ video.date}}</span><br>
                        <span class="tweet-price">{{ video.description }}</span>
                      </div>
                    </div>
                  </div>
            </div>
          </div>
        </div>
      </div> -->
      <div class="col-6">
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
                    <span class="tweet-author">{{ video.likes}}</span><br>
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
          <div v-if="loading">Loading News...</div>
          <div v-else>
            <div v-for="(news, index) in news" :key="index" class="tweet">
              <div class="tweet-content">
                <div class="tweet-header">
                  <img :src="news.source_icon" :alt="news.title" width="70" class="tweet-profile-pic">
                  <div class="tweet-info">
                    <span class="tweet-author"><a :href="news.link" target="_blank">{{ news.title }}</a></span><br>
                    <span class="tweet-author">{{ news.source_name }}</span><br>
                    <span class="tweet-author">{{ news.date }}</span><br>
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
  export default {
    data() {
      return {
        hashtag: '',
        videos: [],
        news: [],
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
        filterData() {
            this.videos = this.videos.filter(video => 
                this.items.some(item => 
                    video.text.toLowerCase().includes(item.text.toLowerCase())
                )
            )
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
  
          const response2 = await fetch('http://localhost:5000/news', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                query: this.hashtag
            })
          });
          const data2 = await response2.json();
          this.news = data2;
  
          this.loading = false;
        } catch (error) {
          console.error('Error fetching videos:', error);
          this.loading = false;
        }
      },
      formatDate(dateString) {
        return new Date(dateString).toLocaleDateString();
      }
    }
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
  </style>
  