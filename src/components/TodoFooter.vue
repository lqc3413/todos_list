<script setup>
 const props = defineProps({
  count: {
    type: Number,
    default: 0
  },
  tabType: {
    type: Number,
    default: 0
  }
})
const emit = defineEmits(['changeTabType'])
const tabClick = (newTabtype) => {
  emit('changeTabType', newTabtype)
}
</script>

<template>
    <div class="tdFooter">
    <span>
    总计：{{ count }}
    </span>
        <div class="tdTabs">
          <a :class="{active:tabType == 0}" @click="tabClick(0)">全部
          </a>
          <a :class="{active:tabType == 1}" @click="tabClick(1)">未完成
          </a>
          <a :class="{active:tabType == 2}" @click="tabClick(2)">已完成
          </a>
        </div>
    </div>
</template>

<style scoped>
.tdFooter {
    background-color: #fff;
    padding: 10px 20px;
    margin: 20px 0;
    display: flex;
    justify-content: space-between;
}

.tdTabs a {
    padding: 0 10px;
    cursor: pointer;
}

.active {
    color: red;
}
</style>
