<template>
  <div :id="treeId" class="treeview" ref="treeview">
    <tree-node
        :key="key(root)"
        v-model="root"
        @navigate="onNavigate"
        @dropitem="dropitem"
    />
  </div>
</template>

<script>
import TreeNode from './TreeNode.vue';
// import './../../lib/jquery/jquery.min.js';
// import './../../lib/contextMenu/jquery.contextMenu.min.js';
// import './../../lib/contextMenu/style.css';
// import './../../html/tree-view-classic.css';

export default {

  components: {
    TreeNode,
  },

  props: {
    modelValue: {
      type: Object,
      required: true
    },

    menuItems: {
      type: Object,
      default: {}
    },
  },

  emits: [
    "menu:click",
    "dropitem",
    "navigate",
  ],

  computed: {
    treeId(){
      return `tree-view-${this.id}`
    },
  },

  data() {
    return {
      id: ~~(Math.random() * 10000),
      root: this.modelValue,
    }
  },

  methods: {

    dropitem(dragNode, toNode){
      if (!dragNode.parent || !dragNode.parent.children || dragNode.parent.children.length === 0){
        return;
      }

      let index = dragNode.parent.children.findIndex((node) => node.id === dragNode.id);
      dragNode.parent.children.splice(index, 1);

      if (!toNode.children){
        toNode.children = [];
      }
      let {nodeData} = dragNode;
      nodeData.focused = false;
      toNode.children.push(nodeData);
    },

    key(node){
      return TreeNode.methods.key(node);
    },

    onNavigate(node, forward, e) {
      TreeNode.methods.onNavigate(node, forward, e);
    },

    create(item){
      //todo fix selected
      this.root.children.push(item);
    },

    isEmptyObject(obj) {
      let name;
      for (name in obj) {
        if (obj.hasOwnProperty(name)) {
          return false;
        }
      }
      return true;
    },

    menuBuild($trigger, e){
      if (this.isEmptyObject(this.menuItems)){
        return false;
      }

      return {
        callback: (key, options) => {
          this.$emit("menu:click", key, options);
          this.$emit("menu:click:" + key, options);
        },
        items: this.menuItems
      };
    }
  },

  async mounted() {
    const tree = this;
    $.contextMenu({
      selector: `#${this.treeId}`,
      build: ($trigger, e) => {
        return this.menuBuild($trigger, e);
      }
    });
  }

}
</script>