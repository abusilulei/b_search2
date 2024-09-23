<template>
  <div id="app">
    <div :class="['container', { 'container-top': searched }]">
      
      <!-- Search Page -->
      <div v-if="!searched" class="homepage-content">
        <h1 class="title">Bilibili 细粒度视频搜索工具</h1>
        <div class="search-modes">
          <label>
            <input type="radio" name="searchMode" value="subtitle" v-model="searchMode"> 字幕搜索
          </label>
          <label>
            <input type="radio" name="searchMode" value="AIsubtitle" v-model="searchMode"> 语音搜索
          </label>
          <label>
            <input type="radio" name="searchMode" value="Visual" v-model="searchMode"> 画面搜索
          </label>
        </div>
      </div>
      <div class="search-bar">
        <button class="home-button" v-if="searched" @click="resetSearch"></button>
        <input
          v-model="searchQuery"
          type="text"
          class="search-input" 
          placeholder="请输入关键词..."
          @keyup.enter="performSearch"
        />
        <button class="search-button" @click="performSearch"></button>
      </div>
      
      <!-- No Results -->
      <div v-if="searched && videos.length === 0" class="no-results">没有找到您搜索的内容！</div>

      <!-- Results Page -->
      <div v-if="searched && videos.length > 0" class="results-container">
        <div class="flex-container">
          <div class="video-section">
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

          <!-- Video List -->
          <div class="video-list">
            <ul>
              <li 
                v-for="(video, index) in videos" 
                :key="index" 
                @click="selectVideo(index)"
                :class="{ 'current-video': index === currentIndex }"
              >
                {{ video.title }}
              </li>
            </ul>
          </div>
        </div>

        <!-- Subtitle Section -->
        <div class="subtitle-section">
          <h3>上下文：{{ currentSubtitle }}</h3>
        </div>

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
      const subtitle = video.subtitles[0];  // Assume first subtitle is displayed
      // return `//player.bilibili.com/player.html?bvid=${video.bvid}&page=${video.page}&t=${Math.floor(subtitle.start_time)}&autoplay=true`;
      return `//player.bilibili.com/player.html?bvid=${video.bvid}&page=${video.page}&t=${video.start_time}&autoplay=true`;
    },

    currentSubtitle() {
      const video = this.videos[this.currentIndex];
      return video.contents || "No matching subtitle";
    },
  },
  
  methods: {
    async performSearch() {
      const response = await fetch('data.json');
      const data = await response.json();

      // Filter videos based on search mode and query
      this.videos = data.reduce((result, video) => {
        if (video.type === this.searchMode) {
          const matchingSubtitles = video.subtitles.filter(subtitle => subtitle.contents.includes(this.searchQuery));
          if (matchingSubtitles.length > 0) {
            matchingSubtitles.forEach(subtitle => {
              result.push({
                ...video,
                start_time: subtitle.start_time,
                contents: subtitle.contents
              });
            });
          }
        }
        return result;
      }, []);

      if (this.videos.length > 0) {
        this.searched = true;
        this.currentIndex = 0;
      } else {
        this.searched = true;
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
    selectVideo(index) {
      this.currentIndex = index;
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
  margin-top: 40px;
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
  margin-bottom: 40px;
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
  background: url('/b_search2/search.png') no-repeat center center;
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
  left: -50px;
  top: 50%;
  transform: translateY(-50%);
  background: url('/b_search2/home.png') no-repeat center center;
  background-size: 20px 20px;
  background-color: #007bff;
  border-radius: 50%;
  border: none;
  width: 40px;
  height: 40px;
  cursor: pointer;
}

.flex-container {
  margin-top: 30px;
  display: flex;
  justify-content: center;
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

.no-results {
  color: red;
  margin-top: 20px;
  font-size: 1.5em;
}




.results-layout {
  display: flex;
  justify-content: flex-start;
  align-items: flex-start;
}

.video-section {
  position: relative;
  width: 70%;
}

.video-section iframe {
  width: 100%;
  height: 400px;
}




.video-list {
  width: 30%;
  max-height: 400px; /* Limit the height */
  overflow-y: auto; /* Enable scrolling */
  margin-left: 40px;
  background-color: #f9f9f9;
  padding: 10px;
  border-radius: 5px;
}

.video-list ul {
  list-style: none;
  padding: 0;

}

.video-list li {
  padding: 10px;
  cursor: pointer;
  color: black;

}

.video-list li.current-video {
  background-color: #007bff;
  color: #fff;
}

.subtitle-section {
  position: flex;
  margin-top: 0px;
  margin-bottom: 20px;
  font-size: 1.5em;
  color: black;
}

.subtitle-section h3 {
  position: absolute;
  left: 40px;
}

</style>
