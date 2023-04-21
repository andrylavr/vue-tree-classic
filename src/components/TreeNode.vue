<template>
  <details
      v-if="children && children.length > 0"
      :open="open"
      :class="{dragstarted}"
  >
    <summary
        :class="{focused, dragovered}"
        @click="click"
        @focus="select"
        @blur="deselect"
        @keydown="keydown"

        @dragstart="dragstart"
        @dragend="dragend"
        @drop="drop"
        @dragenter="dragenter"
        @dragover="dragover"
        @dragover.prevent
        @dragenter.prevent
        @dragleave="dragleave"
        :draggable="draggable"
    >
      <span class="summary-content"><i class="icon" :class="iconClass" v-if="iconClass"></i>{{ name }}</span>
    </summary>

    <div class="folder-content">
      <tree-node
          v-for="(node, index) of children"
          :key="key(node)"
          v-model="children[index]"
          :ref="'subnode-' + node.id"
          @navigate="onNavigate"
          :parent-ids="[...parentIds, id]"
          :parent="this"
          @dropitem="(...args) => $emit('dropitem', ...args)"
      />
    </div>
  </details>

  <div class="leaf" v-else>
    <a
        class="leaf-content"
        :class="{focused, dragovered, dragstarted}"
        href="#"
        @click="click"
        @focus="select"
        @blur="deselect"
        @keydown="keydown"

        @dragstart="dragstart"
        @dragend="dragend"
        @drop="drop"
        @dragenter="dragenter"
        @dragover="dragover"
        @dragleave="dragleave"
        @dragover.prevent
        @dragenter.prevent
        :draggable="draggable"
    >
      <i class="icon" :class="iconClass" v-if="iconClass"></i>
      {{ name }}
    </a>
  </div>
</template>

<script>
let emptyPng = new Image();
emptyPng.src = "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNkYPhfDwAChwGA60e6kgAAAABJRU5ErkJggg==";

let dragNode;

export default {
  name: 'tree-node',

  props: {
    modelValue: {
      type: Object,
      required: true
    },
    parentIds: {
      type: Array,
      default: [],
    },
    parent: {
      type: Object,
      default: null
    },
  },

  emits: [
      "dropitem",
      "update:modelValue",
      "activate",
      "navigate",
  ],


  data() {
    return {
      id: this.modelValue.id,
      focused: !!this.modelValue.focused,
      children: this.modelValue.children,
      name: this.modelValue.name,
      open: this.modelValue.open,
      icon: this.modelValue.icon,
      iconOpen: this.modelValue.iconOpen,
      draggable: "draggable" in this.modelValue ? this.modelValue.draggable : true,

      dragstarted: false,
      dragovered: false,
    }
  },

  watch: {
    focused(){
      this.$emit('update:modelValue', this.nodeData);
    }
  },

  computed: {

    nodeData(){
      let {id, focused, children, name, open, icon, iconOpen, draggable} = this;
      return {id, focused, children, name, open, icon, iconOpen, draggable};
    },

    iconClass() {
      if (this.iconOpen && this.open){
        return this.iconOpen;
      }
      if (this.icon){
        return this.icon;
      }
    },
  },

  methods: {

    key(node){
      return "node-" + node.id;
    },

    click(e) {
      this.focused = true;
      e.preventDefault();
      if (e && e.clientX && e.detail === 1 && e.offsetX >= 0) {
        return;
      }
      this.open = !this.open;
      this.$emit('activate', this);
    },

    select() {
      this.focused = true;
    },

    keydown(e) {
      const {code} = e;

      if (code === 'ArrowLeft' || code === 'ArrowRight') {
        e.preventDefault();
        e.stopPropagation();
      }

      if ($(".context-menu-root:visible").length > 0){
        return;
      }

      if (code === 'ArrowLeft') {
        if (e.target.tagName === "A"){
          e.target
              .parentElement
              .parentElement
              .parentElement
              .firstElementChild.focus();
        }
        if (!this.open && e.target.tagName === "SUMMARY") {
          const parent = e.target.parentElement.parentElement.parentElement.firstElementChild;
          if(parent.tagName === "SUMMARY"){
            parent.focus();
          }
        }
        if (this.open && e.target.tagName === "SUMMARY") {
          this.open = false;
        }
      }

      if (code === 'ArrowRight') {
        if (this.open && e.target.tagName === "SUMMARY" || e.target.tagName === "A"){
          this.$emit('navigate', this, false, e);
        }
        if (!this.open && e.target.tagName === "SUMMARY") {
          this.open = true;
        }
      }

      if (code === 'ArrowUp' || code === 'ArrowDown') {
        const forward = code === 'ArrowUp';
        this.$emit('navigate', this, forward, e);
        e.preventDefault();
      }
    },

    /**
     *
     * @param e {DragEvent}
     */
    dragstart(e){
      this.dragstarted = true;
      dragNode = this;

      e.dataTransfer.setData("id", this.id);
      e.dataTransfer.dropEffect = 'move';
      e.dataTransfer.effectAllowed = 'move';
      e.dataTransfer.setDragImage(emptyPng, 1, 1);
    },

    dragend(){
      this.dragstarted = false;
    },

    /**
     *
     * @param e {DragEvent}
     */
    drop(e){
      this.dragovered = false;
      let whatId = e.dataTransfer.getData("id").toString();
      let toId = this.id.toString();
      if (whatId === toId){
        return;
      }
      for (const parentId of this.parentIds){
        if (whatId === parentId.toString()){
          return;
        }
      }

      this.$emit(
          "dropitem",
          dragNode,
          this
      );

      console.log(`Dropped item ${whatId} to ${toId}`);
    },

    /**
     *
     * @param e {DragEvent}
     */
    dragenter(e){
      this.dragovered = true;
    },

    /**
     *
     * @param e {DragEvent}
     */
    dragover(e){
      // e.target.append(notAllowedPng);
    },

    dragleave(){
      this.dragovered = false;
    },

    deselect() {
      this.focused = false;
    },

    /**
     *
     * @param elem {Element}
     */
    getRoot(elem){
      while (true){
        if (elem.classList.contains("treeview")){
          return elem;
        }
        elem = elem.parentElement;
      }
    },

    /**
     *
     * @param elem {Element}
     */
    isVisible(elem){
      return elem.offsetParent !== null;
    },

    onNavigate(node, forward, e) {
      const root = this.getRoot(e.target);
      const visibleNodes = [...root.querySelectorAll("a, summary")].filter((node) => {
        return this.isVisible(node);
      });

      let index = visibleNodes.findIndex((node) => {
        return node.classList.contains("focused");
      });
      index = forward ? index - 1 : index + 1;
      if (index in visibleNodes){
        visibleNodes[index].focus();
      }
    },

  },
}

</script>