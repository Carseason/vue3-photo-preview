<template>
  <!-- @slot 默认插槽 -->
  <slot />
  <photo-slider 
    :visible="visible" 
    :index="index" 
    :should-transition="shouldTransition" 
    :toggle-overlay="!photoClosable"
    :default-backdrop-opacity="defaultBackdropOpacity" 
    :items="items" 
    :loop="loop" 
    :download-method="downloadMethod"
    @clickPhoto="handleClickPhoto"
    @clickMask="handleClickMask" 
    @changeIndex="updateIndex" 
    @closeModal="handleHide" 
  />
</template>

<script lang='ts'>
import { defineComponent, PropType, provide } from 'vue';
import { updateItemKey, removeItemKey, handleShowKey } from '../symbols';
import useVisible from './useVisible';
import useIndex from './useIndex';
import PhotoSlider from '../PhotoSlider/index.vue';
import { ItemType } from '../types';
import { ref } from 'vue';

export default defineComponent({
  name: 'PhotoProvider',
  components: {
    PhotoSlider
  },
  props: {
    /**
     * 图片点击是否关闭
     */
    photoClosable: {
      type: Boolean,
      default: false,
    },
    /**
     * 背景点击是否关闭
     */
    maskClosable: {
      type: Boolean,
      default: true,
    },
    /**
     * 箭头切换是否需要过渡
     */
    shouldTransition: {
      type: Boolean,
      default: false,
    },
    /**
     * 默认背景透明度
     */
    defaultBackdropOpacity: {
      type: Number,
      default: 1,
    },
    /**
     * 是否循环显示预览图
     */
    loop: {
      type: Boolean,
      default: false,
    },
    /**
     * 下载图片方法，不传使用内置的下载方法
     */
    downloadMethod: {
      type: Function as PropType<(item: ItemType) => void | null>,
      default: null,
    }
  },
  emits: ['indexChange', 'visibleChange'],
  setup(props, { emit }) {
    const onIndexChange = () => {
      emit('indexChange', { index, items, visible });
    };
    const onVisibleChange = () => {
      emit('visibleChange', { index, items, visible });
    };
    const { index, updateIndex } = useIndex(onIndexChange);
    // const { items, updateItem, removeItem } = useItems(index);
    const items = ref<ItemType[]>([]);
    const updateItem = (item: ItemType) => {
      const index = items.value.findIndex(({ key }) => item.key === key);
      if (index > -1) {
        items.value.splice(index, 1, item);
      } else {
        items.value.push(item);
      }
    };
    const removeItem = (key: string) => {
      const nextItems = items.value.filter((item) => item.key !== key);
      const nextEndIndex = nextItems.length - 1;

      items.value = nextItems;
      index.value = Math.max(Math.min(index.value, nextEndIndex), 0);
    };


    const { visible, handleHide, handleShow } = useVisible(items, index, onVisibleChange);

    provide(updateItemKey, updateItem);
    provide(removeItemKey, removeItem);
    provide(handleShowKey, handleShow);
    const handleClickPhoto = () => {
      if (props.photoClosable) {
        handleHide();
      }
    };
    const handleClickMask = () => {
      if (props.maskClosable) {
        handleHide();
      }
    };
    return {
      items,
      updateItem,
      removeItem,
      visible,
      handleHide,
      handleShow,
      index,
      updateIndex,
      handleClickPhoto,
      handleClickMask,
    };
  },
});
</script>

<style lang="scss"></style>
