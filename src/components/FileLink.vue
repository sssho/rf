<template>
  <div class='filelink'>
    <form id='searchBar' class='searchBar'>
      <input
        type='text'
        placeholder='Search words ex. hoge .doc'
        autofocus
        ref='searchInput'
        v-model="searchQuery"
        @keydown.tab.exact.prevent="moveDown()"
        @keydown.shift.tab.exact.prevent="moveUp()"
        @keypress.enter.exact.prevent="fireSelected()"
        @keypress.shift.enter.exact.prevent="fireFolder()"
        @keydown.down.prevent="moveDown()"
        @keydown.up.prevent="moveUp()"
        @keydown.ctrl.n.prevent="moveDown()"
        @keydown.ctrl.p.prevent="moveUp()"
        @keydown.ctrl.r.prevent="fetchHistory()"
        @keydown.ctrl.h.prevent="pseudoBackspace()"
        @keydown.esc.prevent="clearSearchInput()"
        @blur="focusSearchInput()"
      />
    </form>
    <ul>
      <li
        v-for="(item, index) in filteredCandidates"
        :key="index"
        :class="{ selected: index == selectedIndex }"
        :ref="setItemPanelRef"
        @click="runFileWithDefaultApp(item.dir, item.name)"
        @mouseup.right="openFolder(item.dir)"
      >
        <FileLinkItem v-bind:item="item" />
      </li>
    </ul>
    <button @click="fetchHistory()">Reload</button>
    <div class='num-display'>
      {{ currentCandidatesLen }}/{{ candidatesLen }}
    </div>
  </div>
</template>

<script>
import { ref } from 'vue'
import { exec } from 'child_process';
const fs = require('fs');
const path = require('path');
const http = require('http');
const { ipcRenderer } = require('electron')

import FileLinkItem from './FileLinkItem.vue';

const cacheFile = ipcRenderer.sendSync('sync-get-history-filepath')

const icons = {
  // placed under "public" folder
  word: '/img/ms_word_30.png',
  excel: '/img/ms_word_30.png',
  powerpoint: '/img/ms_word_30.png',
  visio: '/img/ms_word_30.png',
  text: '/img/ms_word_30.png',
  image: '/img/ms_word_30.png',
  folder: '/img/FolderClosed_16x.png',
  unknown: '/img/ms_word_30.png',
};

export default {
  name: 'FileLink',
  components: {
    FileLinkItem,
  },
  setup() {
    const searchInput = ref(null)
    let itemPanelRefs = []

    const setItemPanelRef = el => {
      itemPanelRefs.push(el)
    }

    return {
      searchInput,
      itemPanelRefs,
      setItemPanelRef
    }
  },
  data() {
    return {
      selectedIndex: -1,
      searchQuery: '',
      candidates: [
        {
          name: 'empty',
          dir: 'empty',
        },
      ],
      filtered: [],
      candidatesLen: 0,
      currentCandidatesLen: 0,
    };
  },
  computed: {
    filteredCandidates: function () {
      var filterKey = this.searchQuery && this.searchQuery.toLowerCase();

      if (!filterKey) {
        return this.candidates;
      }

      var filtered = this.candidates;
      for (let word of filterKey.trim().split(' ')) {
        filtered = filtered.filter(function (candidate) {
          return [candidate.dir, candidate.name, candidate.fileType].some(
            function (text) {
              return text.toLowerCase().indexOf(word) > -1;
            }
          );
        });
      }
      return filtered;
      //   return this.candidates.filter(function (candidate) {
      //     return [candidate.dir, candidate.name].some(function (text) {
      //       return text.toLowerCase().indexOf(filterKey) > -1;
      //     });
      //   });
    },
  },
  watch: {
    filteredCandidates: {
      handler: function (filterd) {
        this.filtered = filterd;
        this.currentCandidatesLen = filterd.length;
        this.selectedIndex = 0;
      },
    },
  },
  methods: {
    // Debug
    debugPress: function () {
      console.log('press');
    },
    // Search Input
    focusSearchInput: function () {
      this.searchInput.focus();
    },
    clearSearchInput: function () {
      this.searchInput.value = '';
      this.searchQuery = '';
    },
    pseudoBackspace: function () {
      if (this.searchQuery == '') {
        return;
      }
      var tmp = this.searchInput.value.slice(0, -1);
      this.searchInput.value = tmp;
      this.searchQuery = tmp;
    },
    // Cursor
    moveDown: function () {
      if (this.selectedIndex === this.currentCandidatesLen - 1) {
        this.selectedIndex = 0;
      } else {
        this.selectedIndex++;
      }
      this.itemPanelRefs[this.selectedIndex].scrollIntoView({
        block: 'nearest',
      });
    },
    moveUp: function () {
      if (this.selectedIndex === 0) {
        this.selectedIndex = this.currentCandidatesLen - 1;
      } else {
        this.selectedIndex--;
      }
      this.itemPanelRefs[this.selectedIndex].scrollIntoView({
        block: 'nearest',
      });
    },
    // History
    loadCache: function () {
      this.candidates = JSON.parse(fs.readFileSync(cacheFile, 'utf8'));
      this.setIcon(this.candidates);
      this.candidatesLen = this.candidates.length;
    },
    fetchHistory: function () {
      var p = new Promise((resolve, reject) => {
        var options = {
          hostname: 'localhost',
          port: 8888,
          path: '/history',
          method: 'GET',
        };

        var req = http.request(options, function (res) {
          res.setEncoding('utf8');
          res.on('data', function (chunk) {
            resolve(JSON.parse(chunk));
          });
        });

        req.on('error', function (e) {
          reject('problem with request: ' + e.message);
        });

        req.end();
      });

      p.then((result) => {
        this.candidates = result;
        this.setIcon(this.candidates);
        this.candidatesLen = this.candidates.length;
      });
    },
    // Invoke app
    runFileWithDefaultApp: function (dir, name) {
      var fullpath = path.join(dir, name);
      exec('start /b' + ' "pseudo-title" ' + '"' + fullpath + '"');
    },
    openFolder: function (dir) {
      exec('start /b' + ' "pseudo-title" ' + '"' + dir + '"');
    },
    fireSelected: function () {
      var o = this.filtered[this.selectedIndex];
      this.runFileWithDefaultApp(o.dir, o.name);
    },
    fireFolder: function () {
      var o = this.filtered[this.selectedIndex];
      this.openFolder(o.dir);
    },
    // Misc
    setIcon: function (objs) {
      objs.forEach((e) => {
        e['icon'] = icons[e.fileType];
        e.lastAccessed = e.lastAccessed.split('.')[0];
      });
    },
  },
  created: function () {
    this.loadCache();
    this.currentCandidatesLen = this.candidates.length;
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.searchBar {
  margin: 5px;
  margin-bottom: 15px;
  display: flex;
  border: 2px solid #e0e0e0;
  border-radius: 500px;
  padding: 0.15rem 0.25rem 0.15rem 0.75rem;
  align-items: center;
}

.searchBar input[type='text'] {
  flex-grow: 1;
  font-size: 1rem;
  text-align: left;
  border: 0;
  outline: none;
  min-width: 150px;
}

ul {
  margin: 5px;
  padding: 0;
  max-height: 700px;
  overflow: scroll;
  overflow-x: hidden;
  border-bottom: 1px solid #d9e2e6;
  border-top: 1px solid #d9e2e6;
  box-shadow: 0 0 3px #ddd inset;
}

li {
  list-style: none;
  border-top: 1px dashed #e0e0e0;
  background: #fff;
  position: relative;
  cursor: pointer;
}

li:first-child {
  border-top: 1px solid #e0e0e0;
}

.selected {
  background: #f0f0f0 !important;
}

.num-display {
  color: coral;
}

/*スクロールバー全体*/
::-webkit-scrollbar {
  width: 20px;
  background: #f5f5f5;
}

/*スクロールバーの軌道*/
::-webkit-scrollbar-track {
  border-radius: 3px;
  box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.1);
}

/*スクロールバーの動く部分*/
::-webkit-scrollbar-thumb {
  background-color: rgba(0, 0, 50, 0.2);
  border-radius: 2px;
  box-shadow: 0 0 0 1px rgba(255, 255, 255, 0.2);
}
</style>
