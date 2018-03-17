<template>
  <div >
   <div>
     <tree :data="dragData" draggable>

     </tree>
   </div>
  </div>
</template>

<script>
  import tree from './tree'
export default {
  name: 'HelloWorld',
  components:{
    tree
  },
  data () {
    return {
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
          text: '0-1-0'
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
