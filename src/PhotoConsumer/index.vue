<template>
  <div class="PhotoConsumer" 
    @click="handleClick"
  >
    <!-- @slot 默认插槽 -->
    <slot />
  </div>
</template>

<script lang='ts'>
import { defineComponent, inject, onMounted, onUnmounted, toRefs } from 'vue';
import { updateItemKey, removeItemKey, handleShowKey } from '../symbols';
import uniqueId from 'lodash-es/uniqueId';

export default defineComponent({
  name: 'PhotoConsumer',
  props: {
    /**
     * 图片地址
     */
    src: {
      type: String,
      required: true,
    },
    /**
     * 图片介绍
     */
    intro: {
      type: String,
      default: null
    },
    /**
     * 图片下载名称，默认图片名称
     */
    downloadName: {
      type: String,
      default: null
    }
  },
  setup(props) {
    const updateItem = inject(updateItemKey);
    const removeItem = inject(removeItemKey);
    const handleShow = inject(handleShowKey);
    const key = uniqueId();
    const { src, intro, downloadName } = toRefs(props);

    const handleClick = () => {
      handleShow?.(key);
    };
    const getItem = () => ({
      key,
      src: src.value,
      intro: intro.value,
      downloadName: downloadName.value
    });

    onMounted(() => {
      updateItem?.(getItem());
    });
    onUnmounted(() => {
      removeItem?.(key);
    });

    return {
      handleClick
    };
  },
});
</script>

<style lang="scss"></style>
