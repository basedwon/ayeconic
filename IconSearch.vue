<template>
  <v-container class="icon-search-container">
    <v-row>
      <v-col cols="12">
        <v-text-field
          ref="searchField"
          v-model="searchInput"
          label="Semantic Search Icons"
          prepend-inner-icon="mdi-magnify"
          placeholder="Type a concept, e.g., 'money', 'travel', 'network'"
          clearable
          outlined
          autofocus
          hide-details
          class="mb-2"
          :disabled="isEmbedding"
        ></v-text-field>

        <v-expansion-panels flat class="mb-4">
          <v-expansion-panel style="background: transparent; border: 1px solid rgba(255,255,255,0.1);">
            <v-expansion-panel-header class="text--secondary">Advanced Blending</v-expansion-panel-header>
            <v-expansion-panel-content>
              <v-row>
                <v-col cols="12" sm="4">
                  <div class="text-caption text-center">Keyword Weight ({{ keywordWeight }})</div>
                  <v-slider
                    v-model="keywordWeight"
                    min="0" max="2" step="0.1"
                    color="primary" track-color="rgba(255,255,255,0.1)"
                    hide-details
                    @change="triggerSearch"
                  ></v-slider>
                </v-col>
                <v-col cols="12" sm="4">
                  <div class="text-caption text-center">Semantic ML Weight ({{ semanticWeight }})</div>
                  <v-slider
                    v-model="semanticWeight"
                    min="0" max="2" step="0.1"
                    color="primary" track-color="rgba(255,255,255,0.1)"
                    hide-details
                    @change="triggerSearch"
                  ></v-slider>
                </v-col>
                <v-col cols="12" sm="4">
                  <div class="text-caption text-center">Strictness Threshold ({{ similarityThreshold }})</div>
                  <v-slider
                    v-model="similarityThreshold"
                    min="0.1" max="1.5" step="0.1"
                    color="warning" track-color="rgba(255,255,255,0.1)"
                    hide-details
                    @change="triggerSearch"
                  ></v-slider>
                </v-col>
              </v-row>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>

        <v-progress-linear
          v-if="isEmbedding"
          :value="embeddingProgress"
          color="primary"
          height="20"
          striped
          class="mt-2 text-center"
        >
          <template v-slot:default="{ value }">
            <strong class="white--text">Warming up Embedding Engine... {{ Math.ceil(value) }}%</strong>
          </template>
        </v-progress-linear>
      </v-col>
    </v-row>

    <!-- Initial loading -->
    <v-progress-linear
      v-if="!icons.length && !isEmbedding"
      indeterminate
      color="primary"
      class="mt-2"
    ></v-progress-linear>

    <!-- Semantic Search Loader -->
    <v-row v-if="isSearching" class="my-12">
      <v-col cols="12" class="text-center">
        <v-progress-circular indeterminate color="primary" size="64" width="6"></v-progress-circular>
        <div class="mt-4 text--secondary">Searching semantic space...</div>
      </v-col>
    </v-row>

    <!-- Icon Grid -->
    <v-row class="icon-grid" v-else-if="filteredIcons.length">
      <v-col
        v-for="icon in displayedIcons"
        :key="icon.id || icon.name"
        cols="auto"
      >
        <v-tooltip bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-card
              class="d-flex justify-center align-center icon-card pa-2"
              outlined
              hover
              v-bind="attrs"
              v-on="on"
              @click="copyIcon(icon.name)"
            >
              <v-icon large>mdi-{{ icon.name }}</v-icon>
            </v-card>
          </template>
          <span>{{ icon.name }}</span>
        </v-tooltip>
      </v-col>
    </v-row>

    <!-- No Results -->
    <v-row v-else-if="!isEmbedding && !isSearching">
      <v-col cols="12" class="text-center text--secondary">
        No icons found matching "{{ search }}". Try lowering the Strictness Threshold.
      </v-col>
    </v-row>

    <!-- Lazy Load automatically -->
    <v-row v-if="filteredIcons.length > displayedIcons.length && !isEmbedding && !isSearching">
      <v-col cols="12" class="text-center pb-8" v-intersect="onIntersect">
        <v-progress-circular indeterminate color="primary"></v-progress-circular>
      </v-col>
    </v-row>

    <!-- Snackbar -->
    <v-snackbar v-model="snackbar" :timeout="2500" bottom color="success">
      Copied <strong>mdi-{{ copiedIcon }}</strong> to clipboard!
      <template v-slot:action="{ attrs }">
        <v-btn text v-bind="attrs" @click="snackbar = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </v-container>
</template>

<script>
function cosineSimilarity(vecA, vecB) {
  let dotProduct = 0;
  let normA = 0;
  let normB = 0;
  for (let i = 0; i < vecA.length; i++) {
    dotProduct += vecA[i] * vecB[i];
    normA += vecA[i] ** 2;
    normB += vecB[i] ** 2;
  }
  if (normA === 0 || normB === 0) return 0;
  return dotProduct / (Math.sqrt(normA) * Math.sqrt(normB));
}

function getKeywordScore(icon, searchTerm) {
  if (!searchTerm) return 1.0;
  const s = searchTerm;
  let maxScore = 0;

  if (icon.name === s) return 1.0;
  if (icon.name.startsWith(s)) maxScore = Math.max(maxScore, 0.8);
  if (icon.name.includes(s)) maxScore = Math.max(maxScore, 0.5);

  if (icon.aliases) {
    if (icon.aliases.some(a => a === s)) maxScore = Math.max(maxScore, 0.95);
    else if (icon.aliases.some(a => a.startsWith(s))) maxScore = Math.max(maxScore, 0.7);
    else if (icon.aliases.some(a => String(a).includes(s))) maxScore = Math.max(maxScore, 0.4);
  }

  if (icon.tags) {
    if (icon.tags.some(t => t === s)) maxScore = Math.max(maxScore, 0.9);
    else if (icon.tags.some(t => t.startsWith(s))) maxScore = Math.max(maxScore, 0.6);
    else if (icon.tags.some(t => String(t).includes(s))) maxScore = Math.max(maxScore, 0.3);
  }

  return maxScore;
}

export default {
  name: 'IconSearch',
  props: {
    icons: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      searchInput: '',
      search: '',
      keywordWeight: 1.0,
      semanticWeight: 0.8,
      similarityThreshold: 0.4,
      snackbar: false,
      copiedIcon: '',
      isEmbedding: false,
      isSearching: false,
      embeddingProgress: 0,
      page: 1,
      itemsPerPage: 100,
      filteredIcons: [],
    };
  },
  computed: {
    displayedIcons() {
      return this.filteredIcons.slice(0, this.page * this.itemsPerPage);
    }
  },
  mounted() {
    if (typeof window.Emlet === 'undefined' && typeof window.emlet === 'undefined') {
      const script = document.createElement('script');
      script.src = 'https://unpkg.com/emlet';
      script.onload = () => { this.embedIcons(); };
      document.head.appendChild(script);
    } else {
      this.embedIcons();
    }
  },
  watch: {
    searchInput(val) {
      if (this.debounceTimer) clearTimeout(this.debounceTimer);
      
      // If user typed something, show loading state immediately
      if (val && val.trim() !== '') {
        this.isSearching = true;
        this.filteredIcons = [];
      } else {
        this.isSearching = false;
        this.filteredIcons = this.icons;
      }
      
      this.debounceTimer = setTimeout(() => {
        this.search = val;
        this.page = 1;
        this.doSearch();
      }, 300);
    },
    icons(newVal) {
      if (newVal && newVal.length > 0) {
        this.filteredIcons = newVal; // initial load
        this.embedIcons();
      }
    }
  },
  methods: {
    triggerSearch() {
      if (this.search && this.search.trim()) {
        this.page = 1;
        this.doSearch();
      }
    },
    async doSearch() {
      if (this.isEmbedding || !this.search || !this.search.trim()) {
        this.filteredIcons = this.icons;
        this.isSearching = false;
        return;
      }
      
      this.isSearching = true;
      const searchTerm = this.search.toLowerCase().trim();
      let queryVector;
      
      if (this.semanticWeight > 0) {
        try {
          const embedder = typeof window.Emlet !== 'undefined' ? new window.Emlet(96) : window.emlet;
          queryVector = embedder.embed(searchTerm);
        } catch (e) {
          console.warn("Embedding API resolving...");
          // Will just fall back to keyword score if semantic mapping fails
        }
      }
      
      const scoredIcons = [];
      const totalLen = this.icons.length;
      const chunkSize = 1500;
      
      for (let i = 0; i < totalLen; i += chunkSize) {
        const end = Math.min(i + chunkSize, totalLen);
        for (let j = i; j < end; j++) {
          const icon = this.icons[j];
          let simScore = 0;
          let kwScore = 0;
          
          if (queryVector && icon.vector && this.semanticWeight > 0) {
            simScore = Math.max(0, cosineSimilarity(queryVector, icon.vector));
          }
          
          if (this.keywordWeight > 0) {
            kwScore = getKeywordScore(icon, searchTerm);
          }
          
          const finalScore = (kwScore * this.keywordWeight) + (simScore * this.semanticWeight);
          
          if (finalScore >= this.similarityThreshold) {
            scoredIcons.push({ icon, sim: finalScore, originalIndex: j });
          }
        }
        
        // Yield to browser loop so spinner remains smooth
        await new Promise(resolve => setTimeout(resolve, 0));
      }
      
      this.filteredIcons = scoredIcons.sort((a, b) => {
        if (a.sim !== b.sim) return b.sim - a.sim;
        return a.originalIndex - b.originalIndex;
      }).map(item => item.icon);
      
      this.isSearching = false;
    },
    onIntersect(entries, observer, isIntersecting) {
      if (isIntersecting) {
        this.page += 1;
      }
    },
    async embedIcons() {
      const checkEmlet = () => typeof window.Emlet !== 'undefined' || typeof window.emlet !== 'undefined';
      if (!this.icons.length || this.isEmbedding || this.icons[0].vector || !checkEmlet()) return;
      
      this.isEmbedding = true;
      this.embeddingProgress = 0;
      
      const embedder = typeof window.Emlet !== 'undefined' ? new window.Emlet(96) : window.emlet;
      const chunkSize = 200; 
      
      for (let i = 0; i < this.icons.length; i += chunkSize) {
        const chunk = this.icons.slice(i, i + chunkSize);
        chunk.forEach(icon => {
          icon.vector = embedder.embed(icon.name);
        });
        this.embeddingProgress = Math.round((i / this.icons.length) * 100);
        await new Promise(resolve => setTimeout(resolve, 0));
      }
      
      this.embeddingProgress = 100;
      this.isEmbedding = false;
      
      this.$nextTick(() => {
        if (this.$refs.searchField) {
          this.$refs.searchField.focus();
        }
      });
      
      if (this.search && this.search.trim()) {
        this.doSearch();
      }
    },
    copyIcon(name) {
      const iconToCopy = `mdi-${name}`;
      if (navigator.clipboard) {
        navigator.clipboard.writeText(iconToCopy).then(() => {
          this.copiedIcon = name;
          this.snackbar = true;
        }).catch(err => {
          console.error("Failed to copy clipboard", err);
        });
      } else {
        const textarea = document.createElement("textarea");
        textarea.value = iconToCopy;
        document.body.appendChild(textarea);
        textarea.select();
        try {
          document.execCommand("copy");
          this.copiedIcon = name;
          this.snackbar = true;
        } catch (err) {
          console.error("Failed to execute copy", err);
        }
        document.body.removeChild(textarea);
      }
    }
  }
};
</script>

<style scoped>
.icon-search-container {
  max-width: 1200px;
  margin: 0 auto;
}
.icon-grid {
  justify-content: center;
}
.icon-card {
  width: 64px;
  height: 64px;
  cursor: pointer;
  transition: all 0.2s ease-in-out;
  border: 1px solid rgba(57, 255, 20, 0.3) !important;
}
.icon-card:hover {
  transform: translateY(-2px);
  border-color: #39FF14 !important;
  box-shadow: 0 4px 12px rgba(57, 255, 20, 0.2);
}
</style>
