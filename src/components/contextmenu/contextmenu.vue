<template>
  <ul
    v-show="isVisible"
    v-clickoutside="hide"
    class="yb-context-menu"
    :style="style"
    tabindex="-1"
    @contextmenu.capture.prevent>
    <slot></slot>
  </ul>
</template>

<script>
  import Clickoutside from 'element-ui/src/utils/clickoutside';
  const Vue = require('vue')

  export default {
    name: 'yb-context-menu',
    directives: { Clickoutside },
    props:{
      data:{
        type:Array,
        default:()=>[]
      },
      position:{
        type:Object,
        default:()=>({x:0,y:0})
      },
      open:{
        type:Boolean,
        default:false
      },
    },
    mounted() {
      this.$on('yb-menu-item-click', this.handleMenuItemClick);
    },
    watch:{
      open(v){
        if(v){
          this.isVisible = true
        }
      }
    },
    data () {
      return {
        isVisible:false,
        x:0,
        y:0,
        callBack:'',
        context:''
      }
    },
    computed: {
      style () {
        return this.isVisible ? {
          top: this.y - document.body.scrollTop + 'px',
          left: this.x + 'px'
        } : {}
      },
    },
    methods: {
      hide(){
        this.isVisible = false
      },
      handleMenuItemClick(command , item){

        this.hide();
        this.$emit('command', command, item);
        if(this.callBack && typeof this.callBack === 'function'){
          this.callBack(command , this.context)
        }
      }
    }

  }
</script>

<style lang="less"  scoped>
  .yb-context-menu {
    position: fixed;
    z-index: 999;

    padding: 10px 0;
    margin: 5px 0;
    background-color: #fff;
    border: 1px solid #ebeef5;
    border-radius: 4px;
    box-shadow: 0 2px 12px 0 rgba(0,0,0,.1);

  }
  .yb-context-menu:focus {
    outline: none;
  }
</style>
