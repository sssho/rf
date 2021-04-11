<template>
  <div class='filelinkitem'>
    <div class='current-select-marker'>
      <div class='marker'></div>
    </div>
    <div class='icon'>
      <img :src='item.icon'>
    </div>
    <div class='item-text'>
      <div class='title'>
        <!-- <span v-html="result.highlighteds.title">{{ name }}</span> -->
        <span>{{ item.name }}</span>
      </div>
      <div class='path'>
        <!-- <span v-html="result.highlighteds.url">{{ dir }}</span> -->
        <span>{{ item.dir }}</span>
      </div>
      <div class='lastaccess'>
        <span>{{ item.lastAccessed }}</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FileLinkItem',
  props: {
    item: Object,
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.filelinkitem {
  text-decoration: none;
  width: 100%;
  display: flex;
  text-align: left;
}

.filelinkitem:hover {
  background: #f7f7f7 !important;
}

.filelinkitem:hover > .item-text {
    color: #008eff; /* does not work */
    text-decoration: underline;
}

.item-text {
  /* width: 280px; */
  margin-left: 8px;
  padding: 6px 0;
}

.title {
  color: #455a64;
  font-weight: bold;
  font-size: 12px;
  line-height: 1.5em;
  margin-bottom: 3px;
  max-height: 3em;
  overflow: hidden;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
}

.path {
    letter-spacing: 0.05em;
}

.path, .lastaccess {
  color: #888;
  font-size: 11px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.lastaccess {
    margin-top: 2px;
}

.icon {
  position: relative;
  width: 26px;
  margin-left: 5px;
}

img {
  height: 20px;
  width: 20px;
  position: absolute;
  top: 50%;
  left: 50%;
  -webkit-transform: translateY(-50%) translateX(-50%);
  transform: translateY(-50%) translateX(-50%);
  padding: 5px;
  background: #fff;
  box-shadow: 0px 1px 3px #cfd8dc;
}
</style>
