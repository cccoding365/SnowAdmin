<template>
  <div class="container-page">
    <a-input class="search-tree" v-model="searchKey" placeholder="请输入搜索关键词">
      <template #prefix>
        <icon-search />
      </template>
    </a-input>
    <div class="tree-box">
      <!-- TODO: 滚动条边距和宽度需要调整 -->
      <a-scrollbar style="height: 100%; overflow: auto" outer-class="scrollbar">
        <a-tree :data="treeData" :show-line="true" @select="onNode">
          <template #title="node">
            <span class="tree-title">{{ node.title }}</span>
          </template>
          <template #icon="node">
            <SvgIcon name="folder-close" :size="16" v-if="!node.isLeaf && !node.expanded"></SvgIcon>
            <SvgIcon name="folder-open" :size="16" v-if="!node.isLeaf && node.expanded"></SvgIcon>
            <SvgIcon name="txt" :size="16" v-if="node.isLeaf"></SvgIcon>
          </template>
        </a-tree>
      </a-scrollbar>
    </div>
  </div>
</template>

<script setup lang="ts">
import { fileTreeData } from "@/views/file-management/document-library/components/file-tree-data.js";
import { findParentsTailRecursive } from "@/utils";

const emit = defineEmits(["onNode"]);

const onNode = (selectedKeys: Array<string>) => {
  let list = findParentsTailRecursive(treeData.value, selectedKeys[0]);
  emit("onNode", list);
};

const searchKey = ref("");
const treeData = computed(() => {
  if (!searchKey.value) return sourceTree.value;
  return searchData(searchKey.value);
});

// 搜索树
const searchData = (keyword: string) => {
  const loop = (tree: any) => {
    const result: any = []; // 存储搜索结果
    tree.forEach((item: any) => {
      // 匹配节点
      if (item.title.toLowerCase().indexOf(keyword.toLowerCase()) > -1) {
        result.push({ ...item });
      } else if (item.children) {
        // 不匹配，但是有children，深层遍历
        // filterData 存储遍历结果
        const filterData = loop(item.children);
        // 如果在深层遍历中，匹配到
        if (filterData.length) {
          // 将结果存储到children中
          result.push({
            ...item,
            children: filterData
          });
        }
      }
    });
    return result;
  };
  // 开始递归
  return loop(sourceTree.value);
};

const sourceTree = ref(fileTreeData.tree);
</script>

<style lang="scss" scoped>
.container-page {
  box-sizing: border-box;
  padding: $padding;
  height: 100%;
  .search-tree {
    height: 32px;
    margin-bottom: $margin;
  }
  .tree-box {
    height: calc(100% - 32px - $margin);
    .scrollbar {
      height: 100%;
      .tree-title {
        white-space: nowrap;
      }
    }
  }
}
</style>
