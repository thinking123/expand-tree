<template>
  <ul :class="treeCls">
    <li v-for="(item, index) in data" :class="{[`${prefixCls}-treenode-disabled`]: item.disabled,[dropOverCls]: dragOverIndex === index}" @dragover="dragover" @drop="drop(index,$event)" @dragenter="dragenter(index,$event)" @dragleave="dragleave(index,$event)" ref="node">
      <span :class="[`${prefixCls}-switcher`,{[`${prefixCls}-switcher-disabled`]: item.disabled,[`${prefixCls}-switcher-noop`]: item.isLeaf,[`${prefixCls}-switcher_${item.expanded ? 'open' : 'close'}`]: !item.isLeaf}]" @click="setExpand(item.disabled, index)"></span>

      <span :class="prefixCls + 'node-icon' + prefixIcon">
          prefixIcon
      </span>

      <span :title="item.title" :class="selectHandleCls(item)" @click.prevent="setSelect(item.disabled, index)" :draggable="draggable" @dragstart="dragstart(index,$event)" @dragend="dragend"
            v-on:dblclick="setEdit(item.disabled, index)">
        <input type="text" v-model="item.text"  v-if="item.edited" @click.enter="completeEdit(item.disabled, index)"/>
        <span :class="prefixCls + '-title'" v-html="item.title" v-else></span>
      </span>

      <span :class="prefixCls + 'node-icon' + suffixIcon">
        <!--<span :class="prefixCls + '-checkbox-inner'"></span>prefix-->
        suffixIcon
      </span>

      <transition>
        <tree v-if="!item.isLeaf" :prefix-cls="prefixCls" :data="item.children" :clue="`${clue}-${index}`" :checkable="checkable" :class="`${prefixCls}-child-tree-open`" v-show="item.expanded" :draggable="draggable" :async="async"></tree>
      </transition>
    </li>
  </ul>
</template>
<script>
import emitter from '../../mixins/emitter';
import { getOffset } from '../../utils/fn';

export default {
    name: 'Tree',
    mixins: [emitter],
    props: {
      //节点的图标
      prefixIcon:{
        type:String ,
        default:''
      },
      suffixIcon:{
        type:String ,
        default:''
      },

        prefixCls: {
            type: String,
            default: 'yb-tree',
        },
        clue: {
            type: String,
            default: '0',
        },
        data: {
            type: Array,
            default: () => [],
        },
        draggable: {
            type: Boolean,
            default: false,
        },
        canDrop: {
            type: Function,
            default: () => true,
        },
    },
    data: () => ({
        dragIndex: -1,
        dragOverIndex: -1,
        dropPosition: 0,
        dragCrossSameTree: false,
    }),
    computed: {
        treeCls() {
            return [
                `${this.prefixCls}-child-tree`
            ];
        },
        dropOverCls() {
            let res;
            switch (this.dropPosition) {
                case 0:
                    res = 'drag-over';
                    break;
                case 1:
                    res = 'drag-over-gap-bottom';
                    break;
                case -1:
                    res = 'drag-over-gap-top';
                    break;
                default:
            }
            return res;
        },
    },
    watch: {
        data() {
            this.setKey();
            this.preHandle();
        },
    },
    mounted() {
        this.setKey();
        this.preHandle();

        this.$on('nodeSelected', (params) => {
            if (this.clue !== '0') return this.dispatch('Tree', 'nodeSelected', params);
            if (params.status) {
                if (this !== params.origin) {
                    for (let i = 0; i < this.data.length; i++) {
                        this.$set(this.data[i], 'selected', false);
                    }
                }
                this.broadcast('Tree', 'cancelSelected', params.origin);
            }
            this.$emit('select', this.getSelectedNodes());
        });

        this.$on('cancelSelected', (ori) => {
            this.broadcast('Tree', 'cancelSelected', ori);

            if (this !== ori) {
                for (let i = 0; i < this.data.length; i++) {
                    this.$set(this.data[i], 'selected', false);
                }
            }
        });

        //0-1-2-1-2
      //0-2-1
        this.$on('dragdrop', (sourceClue, targetClue, dropPosition) => {
            if (this.clue !== '0') return this.dispatch('Tree', 'dragdrop', [sourceClue, targetClue, dropPosition]);
            // 直接父级是否是同一个
          //but not including indexEnd
          //0-1- , 0-2-
            const sameTree = sourceClue.substr(0, sourceClue.length - 1) === targetClue.substr(0, targetClue.length - 1);
            sourceClue = sourceClue.split('-');//[0,2,1]
            let sourceData = this.data;
            //获取drag的data
            let _sourceData;
            let lastSourceIndex = sourceClue[sourceClue.length - 1] * 1;
            for (let i = 1; i < sourceClue.length - 1; i++) {
                const index = sourceClue[i];
                if (i === 1) {
                    sourceData = sourceData[index];
                } else {
                    sourceData = sourceData.children[index];
                }
            }
            if (sourceClue.length > 2) {
                _sourceData = JSON.parse(JSON.stringify(sourceData.children[lastSourceIndex]));
            } else {
                _sourceData = JSON.parse(JSON.stringify(sourceData[lastSourceIndex]));
            }

            targetClue = targetClue.split('-');
            let targetData = this.data;
            const targetIndex = targetClue[targetClue.length - 1] * 1;

            for (let i = 1; i < targetClue.length - 1; i++) {
                const index = targetClue[i];
                if (i === 1) {
                    targetData = targetData[index];
                } else {
                    targetData = targetData.children[index];
                }
            }
            let canDrop;
            if (targetClue.length > 2) {
                canDrop = this.canDrop(_sourceData, targetData.children[targetIndex], dropPosition);
            } else {
                canDrop = this.canDrop(_sourceData, targetData[targetIndex], dropPosition);
            }
            if (!canDrop) return;

            let sourcePositionChange = false;
//          dropPosition的值有-1(插入到目标节点前面),0(插入到目标节点里面),1(插入到目标节点后面)
            switch (dropPosition) {
                case 0:
                    if (targetClue.length > 2) {
                        targetData = targetData.children[targetIndex];
                    } else {
                        targetData = targetData[targetIndex];
                    }
                    if (targetData.children) {
                        targetData.children.push(_sourceData);
                    } else {
                        this.$set(targetData, 'children', [_sourceData]);
                    }
                    break;
                case 1:
                case -1:
                    const p = targetIndex + (dropPosition === -1 ? 0 : dropPosition);
                    if (targetClue.length > 2) {
                        targetData.children.splice(p, 0, _sourceData);
                    } else {
                        targetData.splice(p, 0, _sourceData);
                    }
                    sourcePositionChange = sameTree && p <= lastSourceIndex;
                    break;
//                case -1:
                default:
            }

            if (sourcePositionChange) lastSourceIndex++;
            if (sourceClue.length > 2) {
                if (sourceData.children.length === 1) {
                    this.$delete(sourceData, 'children');
                } else {
                    sourceData.children.splice(lastSourceIndex, 1);
                }
            } else {
                sourceData.splice(lastSourceIndex, 1);
            }
        });
    },
    methods: {
      setEdit(disabled, index){
        if (!disabled) {
          const edited = !this.data[index].edited;
          this.$set(this.data[index], 'edited', edited);
        }
      },
      completeEdit(disabled, index){
        if (!disabled) {
          const edited = !this.data[index].edited;
          this.$set(this.data[index], 'edited', edited);
        }
      },

        dragstart(index, ev) {
            ev.stopPropagation();
            this.dragIndex = index;
            ev.dataTransfer.setData('dragClue', `${this.clue}-${index}`);
        },
        dragover(ev) {
            ev.preventDefault();
            ev.stopPropagation();
        },
        drop(index, ev) {
            ev.stopPropagation();
            this.dragOverIndex = -1;
            const dragClue = ev.dataTransfer.getData('dragClue');
            const selfClue = `${this.clue}-${index}`;

            // 如果拖拽的对象不是自己的父辈级
            if (!selfClue.startsWith(dragClue)) {
                if (this.clue === '0') {
                    this.$emit('dragdrop', dragClue, selfClue, this.dropPosition);
                } else {
                    this.dispatch('Tree', 'dragdrop', [dragClue, selfClue, this.dropPosition]);
                }
            }
        },
        dragenter(index, ev) {
            ev.preventDefault();
            ev.stopPropagation();
            if (this.dragIndex === index) return;
            if (this.dragOverIndex > -1) this.dragCrossSameTree = true;
            this.dragOverIndex = index;
            const offset = getOffset(this.$refs.node[index]);
            const offsetTop = offset.top;
            const offsetHeight = offset.bottom - offset.top;
            const pageY = ev.pageY;
            const gapHeight = 2;

            if (pageY > offsetTop + offsetHeight - gapHeight) {
                this.dropPosition = 1;
            } else if (pageY < offsetTop + gapHeight) {
                this.dropPosition = -1;
            } else {
                this.dropPosition = 0;
            }
            this.$set(this.data[index], 'expanded', true);
        },
        dragleave(index, ev) {
            ev.stopPropagation();
            if (this.dragIndex === index) return;
            if (this.dragCrossSameTree) {
                this.dragCrossSameTree = false;
            } else {
                this.dragOverIndex = -1;
            }
        },
        dragend(ev) {
            ev.stopPropagation();
            this.dragIndex = -1;
        },
        treeNodeCls(item) {
            return {
                [`${this.prefixCls}-treenode-disabled`]: item.disabled,
            };
        },
        checkboxCls(item) {
            return [
                `${this.prefixCls}-checkbox`,
                {
                    [`${this.prefixCls}-checkbox-disabled`]: item.disabled || item.disableCheckbox,
                    [`${this.prefixCls}-checkbox-checked`]: item.checked && item.childrenCheckedStatus === 2,
                    [`${this.prefixCls}-checkbox-indeterminate`]: item.checked && item.childrenCheckedStatus === 1,
                },
            ];
        },
        selectHandleCls(item) {
            const wrap = `${this.prefixCls}-node-content-wrapper`;

            return [
                wrap,
                `${wrap}-normal`,
                {
                    [`${this.prefixCls}-node-selected`]: !item.disable && item.selected,
                    draggable: this.draggable,
                },
            ];
        },
        setKey() {
            for (let i = 0; i < this.data.length; i++) {
                this.data[i].clue = `${this.clue}-${i}`;
            }
        },
        preHandle() {
            for (const [i, item] of this.data.entries()) {
                if (!item.children) {
                    this.$set(item, 'isLeaf', true);
                    continue;
                }

                this.$set(item, 'isLeaf', false);
            }
        },
        async setExpand(disabled, index) {
            if (!disabled) {
                const expanded = !this.data[index].expanded;
                this.$set(this.data[index], 'expanded', expanded);
            }
        },
        setSelect(disabled, index) {
            if (!disabled) {
                const selected = !this.data[index].selected;

                if (!selected) {
                    this.$set(this.data[index], 'selected', selected);
                } else {
                    for (let i = 0; i < this.data.length; i++) {
                        if (i === index) {
                            this.$set(this.data[i], 'selected', true);
                        } else {
                            this.$set(this.data[i], 'selected', false);
                        }
                    }
                }

                if (this.clue === '0') {
                    this.$emit('nodeSelected', { origin: this, status: selected });
                } else {
                    this.dispatch('Tree', 'nodeSelected', { origin: this, status: selected });
                }
            }
        },
        setCheck(disabled, index) {
            if (disabled) return;

            const checked = !this.data[index].checked;
            this.$set(this.data[index], 'checked', checked);
            this.$set(this.data[index], 'childrenCheckedStatus', checked ? 2 : 0);
            if (this.clue === '0') {
                this.$emit('childChecked', { origin: this, clue: this.clue });
            } else {
                this.dispatch('Tree', 'childChecked', { origin: this, clue: this.clue });
            }
            this.broadcast('Tree', 'parentChecked', { status: checked, clue: `${this.clue}-${index}` });
        },
        getNodes(data, opt) {
            data = data || this.data;
            let res = [];

            for (const node of data) {
                let tmp = true;
                for (const [key, value] of Object.entries(opt)) {
                    if (node[key] !== value) {
                        tmp = false;
                        break;
                    }
                }
                if (tmp) {
                    res.push(node);
                }
                if (node.children && node.children.length) {
                    res = res.concat(this.getNodes(node.children, opt));
                }
            }
            return res;
        },
        getSelectedNodes() {
            return this.getNodes(this.data, { selected: true });
        },
        edit(path, action, data) {
            path = path.split('-');
            const isTopNode = path.length === 2;

            let node = this.data;
            const lastIndex = path.pop();

            if (!isTopNode) node = node[path[1]];
            path.splice(0, 2);

            for (const i of path) {
                node = node.children[i];
            }
            switch (action) {
                case 'delete':
                    if (isTopNode) {
                        node.splice(lastIndex, 1);
                    } else {
                        node.children.splice(lastIndex, 1);
                    }
                    break;
                case 'add':
                    let child;
                    if (isTopNode) {
                        child = node[lastIndex];
                    } else {
                        child = node.children[lastIndex];
                    }
                    if (child.children) {
                        child.children.push(data);
                    } else {
                        this.$set(child, 'children', [data]);
                    }
                    break;
                case 'edit':
                    node = isTopNode ? node[lastIndex] : node.children[lastIndex];

                    for (const [key, val] of Object.entries(data)) {
                        node[key] = val;
                    }
                    break;
                default: break;
            }
        },
        editNode(path, data) {
            this.edit(path, 'edit', data);
        },
        addNode(path, data) {
            if (path === '0') return this.data.push(data);
            this.edit(path, 'add', data);
        },
        delNode(path) {
            this.edit(path, 'delete');
        },
    },
};
</script>
