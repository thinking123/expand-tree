<template>
  <div >
   <div>
     <tree :data="dragData">

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
        title: '0-0',
        expanded: true,
        children: [{
          title: '0-0-0',
          expanded: true,
          children: [{
            title: '0-0-0-0',
          }, {
            title: '0-0-0-1',
          }]
        }, {
          title: '0-0-1',
          children: [{
            title: '0-0-1-0'
          }]
        }]
      }, {
        title: '0-1',
        children: [{
          title: '0-1-0'
        }, {
          title: '0-1-2'
        }]
      }, {
        title: '0-2',
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
