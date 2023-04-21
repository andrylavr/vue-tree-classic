<!--This is example app -->
<template>
  <div class="main">
    <h3>Test Tree</h3>

    <button @click="create">
      Create
    </button>

    <button @click="qTest">
      qTest

      {{quitDisabled}}
    </button>

    <div>
      <div class="col">
        <tree-classic
            v-model="treeData"
            v-model:selected="selectedItem"

            ref="tree"
            @menu:click="menuClick"
            @menu:click:edit="menuClickEdit"
            :menu-items="treeMenuItems"
        />
      </div>
      <div class="col">
        <vue-json-pretty
            :data="treeData"
            :show-icon="true"
            :show-length="true"
            :show-line="true"
            :show-double-quotes="false"
        />
      </div>
    </div>


  </div>
</template>

<style>

  .treeview{
    font-size: 13pt;
    font-family: 'Roboto', sans-serif !important;
    font-weight: 300 !important;
  }

  .col{
    /*float: left;*/
    width: 49%;
  }

  button{
    margin: 10px 5px;
  }

  .main{
    padding: 10px 40px 30px 40px;
    top: 200px;
    left: 25vw;
    /*border: 1px dotted black;*/
    /*position: fixed;*/
    width: 50vw;
  }

  .treeview{
    font-family: system-ui, Ubuntu, "Droid Sans", sans-serif;
  }
</style>

<script>
import TreeClassic from "./components/TreeClassic.vue";
import VueJsonPretty from 'vue-json-pretty';
import 'vue-json-pretty/lib/styles.css';

export default {
  components: {
    TreeClassic,
    VueJsonPretty,
  },

  computed:{
    treeMenuItems(){
      return {
        edit: {name: "Edit", icon: "fa-globe"},
        edit2: {name: "Edit Orig", icon: "edit"},
        cut: {name: "Cut", icon: "cut", callback: this.cut},
        copy: {name: "Copy", icon: "copy"},
        paste: {name: "Paste", icon: "paste"},
        delete: {name: "Delete", icon: "delete"},
        sep1: "---------",
        quit: {
          disabled: this.quitDisabled,
          icon: "fa-tree",
          name: "Quit",
        }
      };
    }
  },

  methods: {
    create(){
      this.$refs.tree.create({
        id: ~~((new Date).getTime()/1000),
        icon: "fa fa-file",
        name: "New Item"
      });
    },

    qTest(){
      this.quitDisabled = !this.quitDisabled;
    },

    cut(){
      console.log(arguments);
    },

    menuClick(key, opts){
      console.log({key, opts});
    },

    menuClickEdit(opts){
      console.log({opts});
      alert("menuClickEdit!");
    },
  },


  data() {
    return {
      quitDisabled: true,

      selectedItem: null,

      treeData: {
        name: "My Project",
        icon: "fa fa-folder",
        open: true,
        id: 0,
        children: [
          {
            "id": 1,
            "name": "users.txt",
            "icon": "fa fa-users"
          },
          {
            "id": 11,
            "name": "Empty Folder",
            "icon": "fa fa-folder",
            "children": [],
          },
          {
            "id": 2,
            "name": "Books",
            "icon": "fa fa-book",
            "children": [
              {
                "id": 3,
                "name": "Neptune",
                "icon": "fa fa-book"
              }
            ]
          },

          {
            "id": 5,
            "name": "Vehicles",
            "open": false,
            icon: "fa fa-folder",
            iconOpen: "fa fa-folder-open",
            "children": [
              {
                "id": 123,
                "name": "Trucks 123",
                "icon": "fa fa-truck",
                "children": [
                  {
                    "id": 123101,
                    "name": "Mars",
                    "icon": "fa fa-globe-w",
                    "children": [
                      {
                        "id": 1231011,
                        "name": "Sub Mars 1",
                        "icon": "fa fa-file",
                      },
                      {
                        "id": 1231012,
                        "name": "Sub Mars 2",
                        "icon": "fa fa-file",
                      }
                    ]
                  },
                  {
                    "id": 123102,
                    "name": "Nepo",
                    "icon": "fa fa-globe-w",
                  },
                ]
              },
              {
                "id": 23,
                "name": "Cars",
                "icon": "fa fa-car"
              },
              {
                "id": 24,
                "name": "Vans",
                "icon": "fa fa-car"
              },
              {
                "id": 25,
                "name": "Bikes",
                "icon": "fa fa-car"
              },
              {
                "id": 34,
                "name": "Trucks",
                "icon": "fa fa-truck",
                "children": [
                  {
                    "id": 101,
                    "name": "Mars",
                    "icon": "fa fa-globe-w"
                  },
                  {
                    "id": 102,
                    "name": "Nepo",
                    "icon": "fa fa-globe-w",
                  },
                ]
              },
            ]
          }
        ]
      },
    }
  },
}
</script>