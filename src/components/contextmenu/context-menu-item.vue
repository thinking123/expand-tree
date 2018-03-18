<template>
  <li
    class="el-dropdown-menu__item"
    :class="{
      'is-disabled': disabled,
      'el-dropdown-menu__item--divided': divided
    }"
    @click="handleClick"
    :aria-disabled="disabled"
    :tabindex="disabled ? null : -1"
  >
    <slot></slot>
  </li>
</template>
<style lang="less" scoped>
  .el-dropdown-menu__item {
    list-style: none;
    line-height: 36px;
    padding: 0 20px;
    margin: 0;
    font-size: 14px;
    color: #606266;
    cursor: pointer;
    outline: none;
    display: list-item;

    &:not(.is-disabled):hover{
      background-color: #ecf5ff;
      color: #66b1ff;
    }

    &.is-disabled{
      cursor: default;
      color: #bbb;
      pointer-events: none;
    }
  }


  .el-dropdown-menu__item--divided {
    position: relative;
    margin-top: 6px;
    border-top: 1px solid #ebeef5;
  }
</style>
<script>
  import Emitter from '../../mixins/emitter';

  export default {
    name: 'yb-context-menu-item',

    mixins: [Emitter],

    props: {
      command: {},
      disabled: Boolean,
      divided: Boolean
    },

    methods: {
      handleClick(e) {
        this.dispatch('yb-context-menu', 'yb-menu-item-click', [this.command, this]);
      }
    }
  };
</script>
