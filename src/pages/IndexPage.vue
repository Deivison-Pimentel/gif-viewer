<template>
  <div>
    <div class="header sticky top-0 bg-black shadow-md z-10">
      <div>
        <div class="search-bar flex items-center">
          <input v-model="searchTerm" type="text" placeholder="Buscar Gif..."
            class="px-4 py-2 w-3/4 border border-gray-300 rounded-l-full focus:outline-none focus:ring focus:border-blue-300" />
          <button @click="searchGifs"
            class="px-4 py-2 bg-blue-500 text-white rounded-r-full hover:bg-blue-600 focus:outline-none focus:ring focus:border-blue-300">
            Buscar
          </button>
        </div>
      </div>
    </div>
    <div class="gif-container mt-16">
      <div class="gif-grid">
        <div v-for="(gif, index) in gifs" :key="gif.id" class="gif-item" :style="{ marginTop: shouldAddMargin(index) ? '2px' : 0 }" @click="showFullScreen(gif)">
          <img :src="gif.images.original.url" alt="GIF"
            class="object-cover border border-gray-300 rounded-md cursor-pointer" />
        </div>
      </div>
      <div v-if="loading" class="text-center mt-4">Carregando...</div>
      <div v-if="error" class="text-center mt-4">{{ error }}</div>
    </div>

    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <span class="close" @click="closeModal">&times;</span>
        <img :src="selectedGif.images.original.url" alt="GIF" class="modal-img" />
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      gifs: [],
      loading: false,
      error: '',
      page: 0,
      limit: 20,
      threshold: 20,
      searchTerm: '',
      showModal: false,
      selectedGif: null,
    };
  },
  mounted() {
    this.fetchGifs();
    window.addEventListener('scroll', this.handleScroll);
    window.addEventListener('resize', this.handleResize);
  },
  beforeUnmount() {
    window.removeEventListener('scroll', this.handleScroll);
    window.removeEventListener('resize', this.handleResize);
  },
  methods: {
    async fetchGifs() {
      try {
        this.loading = true;
        const response = await axios.get('https://api.giphy.com/v1/gifs/trending', {
          params: {
            api_key: 'nbEAnJap9LlY6T3OJzjYSLu6fT2gxIIi',
            limit: this.limit,
            offset: this.page * this.limit,
          },
        });
        this.gifs = this.gifs.concat(response.data.data);
        this.page++;
        this.loading = false;
      } catch (error) {
        this.error = 'Falha ao carregar os GIFs. Por favor, tente novamente mais tarde.';
        this.loading = false;
      }
    },
    async searchGifs() {
      try {
        this.loading = true;
        const response = await axios.get('https://api.giphy.com/v1/gifs/search', {
          params: {
            api_key: 'nbEAnJap9LlY6T3OJzjYSLu6fT2gxIIi',
            q: this.searchTerm,
            limit: this.limit,
            offset: 0,
          },
        });
        this.gifs = response.data.data;
        this.page = 1;
        this.loading = false;
      } catch (error) {
        this.error = 'Falha ao buscar os GIFs. Por favor, tente novamente mais tarde.';
        this.loading = false;
      }
    },
    handleScroll() {
      if (window.innerHeight + window.scrollY >= document.body.offsetHeight - this.threshold) {
        this.fetchGifs();
      }
    },
    handleResize() {

      this.fetchGifs();
    },
    shouldAddMargin(index) {
      if (index === 0) return false;

      const currentHeight = this.gifs[index].images.original.height;
      const prevHeight = this.gifs[index - 1].images.original.height;

      return currentHeight < prevHeight;
    },
    showFullScreen(gif) {
      this.selectedGif = gif;
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
    },
  },
};
</script>

<style scoped>
.gif-container {
  max-width: 1120px;
  margin: 0 auto;
}

.gif-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(255px, 1fr));
  grid-gap: 5px;
}

.gif-item {
  margin-top: 2px;
}

.gif-item img {
  width: 100%;
  height: auto;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.search-bar {
  display: flex;
  align-items: center;
}

.search-bar input {
  max-width: 400px;
}

.header {
  position: sticky;
  top: 0;
  font-size: 30px;
  background-color: #ffffff;
  z-index: 10;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.modal {
  display: block;
  position: fixed;
  z-index: 999;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  overflow: auto;
  background-color: rgba(0, 0, 0, 0.9);
}

.modal-content {
  margin: 5% auto;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
}

.modal-img {
  width: auto;
  max-width: 90%;
  max-height: 90vh;
}

.close {
  color: #fff;
  position: absolute;
  top: 15px;
  right: 35px;
  font-size: 40px;
  font-weight: bold;
  cursor: pointer;
}
</style>
