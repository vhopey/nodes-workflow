<template>
  <div class="search_wrapper">
    <input
      class="search_input"
      v-model="searchValue"
      type="text"
      placeholder="Add new node..."
    />
    <ul class="search_list">
      <li
        class="search_element"
        v-for="node in searchNodes"
        :key="node.label"
        :value="node"
        @mousedown="select(node)"
      >
        <div class="search_node_results">
          {{ node.label }}
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
import { v4 as uuidv4 } from 'uuid';

export default {
  name: 'SearchNodes',
  props: {
    // improve: get list from the server
    nodeList: Array,
    onCreate: Function,
  },
  data() {
    return {
      searchValue: '',
    };
  },
  computed: {
    searchNodes() {
      if (this.searchValue) {
        const regexp = new RegExp(this.searchValue, 'i');
        return this.nodeList.filter((node) => regexp.test(node.label));
      }
      return this.nodeList;
    },
  },
  methods: {
    select(node) {
      this.searchValue = '';
      this.onCreate({
        ...node,
        id: uuidv4(),
      });
    },
  },
};
</script>

<style scoped>
.search_wrapper {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.search_input {
  width: 300px;
  padding: 10px;
  border: 1px solid gray;
}

.search_list {
  display: none;
  position: fixed;
  top: 64px;
  z-index: 2;
  width: 320px;
  max-height: 120px;
  overflow: scroll;
  background-color: white;
  border: 1px solid #b7b6b6;
  border-top: 0px;
}

.search_element {
  display: flex;
  padding: 10px;
  cursor: pointer;
  border-bottom: 1px solid #b7b6b6;
}

.search_list > li:last-child {
  border-bottom: 0px;
}

.search_element:hover {
  background-color: rgba(0, 0, 0, 0.05);
}

.search_wrapper:focus-within .search_list {
  display: block;
}

.search_node_results {
  display: flex;
  justify-content: center;
}
</style>
