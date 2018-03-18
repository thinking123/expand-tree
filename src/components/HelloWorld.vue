<template>
  <div >
    <tool-panel :tools="tools">

    </tool-panel>
   <div>
     <tree :data="dragData" draggable ref="tree">

     </tree>
   </div>
    <context-menu :open="openMenu" @command="handleCommand">
      <context-menu-item command="edit">
        edit
      </context-menu-item>
      <context-menu-item command="create" divided>
        create
      </context-menu-item>
      <context-menu-item command="select" disabled>
        select
      </context-menu-item>
    </context-menu>
  </div>
</template>

<script>
  import tree from './tree'
  import contextMenu from './contextmenu/contextmenu.vue'
  import contextMenuItem from './contextmenu/context-menu-item.vue'
  import Vue from 'vue';
  import toolPanel from './toolanel/tool-panel.vue'
export default {
  name: 'HelloWorld',
  components:{
    tree,
    contextMenu,
    contextMenuItem,
    toolPanel
  },
  mounted(){
    let cm = new Vue(contextMenu).$mount()
    this.$refs.tree.$children.push(cm)
  },
  methods:{
    handleCommand(command){
      if(command == 'create'){

      }
    },
    select(){
      alert('sel')
    },
    showmenu([item ,e]){
      this.openMenu = true
      console.log(e)
    },
    edit(){
      alert("edit")
    }
  },
  data () {
    return {
      tools:[
        {icon:'fa-square-o' , text:'Box'},
        {icon:'fa-text-width' , text:'text'},
        {icon:'fa-picture-o' , text:'Image'},
        {icon:'fa-bandcamp' , text:'Icons'},
        {icon:'fa-columns' , text:'Slot'},
      ],
      openMenu:false,
      dragData: [{
        text: '0-0',
        expanded: true,
        children: [{
          text: '0-0-0',
          expanded: true,
          children: [{
            text: '0-0-0-0',
            isError:true
          }, {
            text: '0-0-0-1',
          }]
        }, {
          text: '0-0-1',
          children: [{
            text: '0-0-1-0'
          }]
        }]
      }, {
        text: '0-1',
        children: [{
          text: '0-1-0',
          disabled:true
        }, {
          text: '0-1-2'
        }]
      }, {
        text: '0-2',
      }],
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
