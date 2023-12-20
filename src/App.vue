<template>
  <div id="app">
    <SearchNodes :nodeList="allNodes" :onCreate="handleCreate" />
    <NodesWorkflow
      :activeNodes="activeNodes"
      :onDeleteNode="deleteActiveNode"
    />
  </div>
</template>

<script>
import { fabric } from 'fabric';
import NodesWorkflow from './components/NodesWorkflow.vue';
import SearchNodes from './components/SearchNodes.vue';
import { mockNodes } from './mock.js';

export default {
  name: 'App',
  components: {
    NodesWorkflow,
    SearchNodes,
  },
  // improve: get data from the server
  data() {
    return {
      activeNodes: [],
      allNodes: mockNodes,
    };
  },
  methods: {
    handleCreate(node) {
      // improve: save node with correct data for server
      this.activeNodes = [...this.activeNodes, node];
    },

    deleteActiveNode(id) {
      this.activeNodes = this.activeNodes.filter((item) => item.id !== id);
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin-top: 20px;
}

input {
  all: unset;
}

ul {
  all: unset;
}
</style>
