<template>
  <div class="header">
    <span v-for="tool in tools" draggable @dragstart="dragstart($event , tool)"
          @dragend="dragend">
          <i  class="fa" :class="tool.icon">

      </i>
      {{tool.text}}
    </span>

  </div>
</template>
<script>
  export default {
    props:{
      tools:{
        type:Array,
        default:()=>[]
      }
    },
    methods:{
      dragstart(ev , tool) {
        ev.stopPropagation();
        ev.dataTransfer.setData('dragTool', tool);
      },
      dragend(ev) {
        ev.stopPropagation();
      },
    }
  }
</script>
