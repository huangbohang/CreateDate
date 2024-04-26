
<template>
  <div>
    <a-typography-text
      >按规律生成日期👉
      <a-typography-text type="primary" @click="helpVoid">
        查看教程
      </a-typography-text>
    </a-typography-text>
    <div class="row-between-center m-b-10">
      <a-typography-text>添加生成规律</a-typography-text>
      <a-button type="primary">新增</a-button>
    </div>
    <a-table :columns="columns"></a-table>

    <addRulePop></addRulePop>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch, onUnmounted } from "vue";
import addRulePop from "./addRulePop.vue";
import dayjs from "dayjs";
const columns=ref([ {
        title: '规律',
        dataIndex: 'rule',
      },{
        title: '时间',
        dataIndex: 'time',
      },{
        title: '条数',
        dataIndex: 'num',
      }])



</script>

<style>
.labelCss {
  width: 70px;
  text-align: center;
  flex-shrink: 0;
}
</style>
