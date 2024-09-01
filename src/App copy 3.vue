<template>
  <div id="app">
    <div :class="['container', { 'container-top': searched }]">
      <div v-if="!searched" class="homepage-content">
        <h1 class="title">Bilibili Video Search</h1>
        <div class="search-modes">
          <label>
            <input type="radio" v-model="searchMode" value="subtitle" />
            Search by Subtitle
          </label>
          <label>
            <input type="radio" v-model="searchMode" value="visual" />
            Search by Visual Content
          </label>
        </div>
      </div>
      <div class="search-bar">
        <button class="home-button" v-if="searched" @click="resetSearch"></button>
        <input
          v-model="searchQuery"
          type="text"
          class="search-input" 
          placeholder="Enter keywords..."
          @keyup.enter="performSearch"
        />
        <button class="search-button" @click="performSearch"></button>
      </div>
      <div v-if="searched" class="video-container">
        <iframe :src="currentVideoUrl" frameborder="no" allowfullscreen="true"></iframe>
        <button
          @click="prevVideo"
          :disabled="currentIndex === 0"
          class="nav-button prev"
          :class="{ disabled: currentIndex === 0 }"
        >
          &#9664;
        </button>
        <button
          @click="nextVideo"
          :disabled="currentIndex === videos.length - 1"
          class="nav-button next"
          :class="{ disabled: currentIndex === videos.length - 1 }"
        >
          &#9654;
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      searchQuery: '',
      searchMode: 'subtitle',
      searched: false,
      videos: [],
      currentIndex: 0,
    };
  },
  computed: {
    currentVideoUrl() {
      const video = this.videos[this.currentIndex];
      return `//player.bilibili.com/player.html?bvid=${video.bvid}&page=${video.page}&t=${Math.floor(video.start_time)}&autoplay=true`;
    },
  },
  methods: {
    async performSearch() {
      const response = await fetch('data.json');
      const data = await response.json();

      this.videos = data.filter(video => video.contents.includes(this.searchQuery));
      if (this.videos.length > 0) {
        this.searched = true;
        this.currentIndex = 0;
      }
    },
    resetSearch() {
      this.searched = false;
      this.searchQuery = '';
      this.videos = [];
    },
    prevVideo() {
      if (this.currentIndex > 0) {
        this.currentIndex--;
      }
    },
    nextVideo() {
      if (this.currentIndex < this.videos.length - 1) {
        this.currentIndex++;
      }
    },
  },
};
</script>

<style>
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
  background: linear-gradient(to right, #e0f7fa, #80b2ea); /* Light blue gradient background */
}

#app {
  max-height: 100%;
  width: 100%;
  text-align: center;
  padding: 20px;
  box-sizing: border-box;
  display: flex; 
  justify-content: center;
  align-items: center; 
}

.container {
  background: rgba(255, 255, 255, 0.8); /* Semi-transparent white background */
  border-radius: 15px;
  padding: 40px;
  max-width: 900px;
  width: 100%;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.3);
  transition: transform 0.5s ease, max-width 0.5s ease;
  position: relative;
}

.container-top {
  position: absolute;
  overflow: visible;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  max-width: 100%;
  width: calc(100% - 50px);
}

.homepage-content {
  margin-bottom: 20px;
}

.title {
  font-size: 3em;
  margin-bottom: 20px;
  color: black;
  font-family: 'Arial', sans-serif;
  font-weight: bolder;
}

.search-modes {
  margin-bottom: 20px;
  color:rgba(0, 0, 0, 0.33)
}

.search-modes label {
  margin-right: 20px;
  font-size: 1.2em;
}

.search-bar {
  position: relative;
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
}

input[type="text"] {
  width: calc(100% - 30px);
  padding: 10px 50px 10px 60px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 20px;
  box-sizing: border-box;
  background-color: white;
}

.search-button {
  position: absolute;
  right: 15px;
  top: 50%;
  transform: translateY(-50%);
  background: url('search.png') no-repeat center center;
  background-size: 20px 20px;
  background-color: #007bff;
  border-radius: 50%;
  border: none;
  width: 40px;
  height: 40px;
  cursor: pointer;
}

.home-button {
  position: absolute;
  left: -30px;
  top: 50%;
  transform: translateY(-50%);
  background: url('home.png') no-repeat center center;
  background-size: 20px 20px;
  background-color: #007bff;
  border-radius: 50%;
  border: none;
  width: 40px;
  height: 40px;
  cursor: pointer;
}

.video-container {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
  min-height: 430px; /* Match the height of the iframe */
}

iframe {
  width: 100%;
  max-width: 640px;
  height: 430px;
  border-radius: 15px;
  border: none;
}

.nav-button {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  font-size: 40px;
  background-color: transparent;
  border: none;
  cursor: pointer;
  z-index: 10;
  padding: 10px;
  color: #007bff; /* Blue color for navigation buttons */
}

.nav-button.prev {
  left: -40px; /* Adjust this value if needed */
}

.nav-button.next {
  right: -40px; /* Adjust this value if needed */
}

.nav-button.disabled {
  color: gray;
  cursor: not-allowed;
}

</style>
