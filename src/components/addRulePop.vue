
<template>
  <!-- <a-modal> -->
  <div class="grid-one m-t-10">
    <div class="row-start-center">
      <a-typography-text class="labelClass">选择规律</a-typography-text>
      <a-radio-group v-model="ruleType">
        <a-radio value="week">周</a-radio>
        <a-radio value="day">日</a-radio>
      </a-radio-group>
    </div>
    <!-- 按照周的规则 -->
    <div v-if="ruleType == 'week'" class="row-start-center">
      <a-typography-text class="labelClass">选择周</a-typography-text>
      <a-select
        :options="optionsWeek"
        multiple
        placeholder="选择周几"
      ></a-select>
    </div>
    <!-- 日期按照相隔 -->
    <div v-if="ruleType == 'day'" class="row-start-center">
      <a-typography-text class="labelClass">间隔天数</a-typography-text>
      <a-input-number
        mode="button"
        size="large"
        v-model="spaceDayNum"
        :min="1"
      ></a-input-number>
    </div>
    <div class="row-start-center">
      <a-typography-text class="labelClass">时间点</a-typography-text>
      <a-radio-group v-model="is_time_range">
        <a-radio :value="false">时间点</a-radio>
        <a-radio :value="true">时间段</a-radio>
      </a-radio-group>
    </div>
    <div class="row-start-center">
      <a-typography-text class="labelClass">时间</a-typography-text>
      <a-time-picker v-if="is_time_range" type="time-range" class="flex-grow" />
      <a-time-picker v-else class="flex-grow" />
    </div>
    <div class="row-start-center">
      <a-typography-text class="labelClass">生产方式</a-typography-text>
      <a-radio-group v-model="createType">
        <a-radio value="create_maxNum">条数</a-radio>
        <a-radio value="create_dateRange">日期范围</a-radio>
      </a-radio-group>
    </div>
    <!-- 生成条数 -->
    <div v-if="createType == 'create_maxNum'" class="row-start-center">
      <a-typography-text class="labelClass">天数</a-typography-text>
      <a-input-number
        mode="button"
        size="large"
        v-model="create_maxNum"
        :min="1"
      ></a-input-number>
    </div>
    <!-- 日期范围 -->
    <div v-if="createType == 'create_dateRange'" class="row-start-center">
      <a-typography-text class="labelClass">日期范围</a-typography-text>
      <a-range-picker
        v-model="create_dateRange"
        class="flex-grow"
      />
    </div>
  </div>
  <!-- </a-modal> -->
</template>
<script setup>

import { cloneDeep } from "lodash";
import { computed, ref, watch, watchEffect } from "vue";
import { Message } from "@arco-design/web-vue";
const ruleType = ref("week");
const spaceDayNum = ref(1); //间隔天数
const is_time_range = ref(false); //false 时间端  true
const createType = ref("create_maxNum"); //create_maxNum create_dateRange
const create_maxNum = ref(1);//最大条数
const create_dateRange = ref([]);//日期范围
 
const optionsWeek = ref([
  { label: "周一", value: "1" },
  { label: "周二", value: "2" },
  { label: "周三", value: "3" },
  { label: "周四", value: "4" },
  { label: "周五", value: "5" },
  { label: "周六", value: "6" },
  { label: "周日", value: "7" },
]);
</script>

<style>
.labelClass {
  width: 80px;
  flex-shrink: 0;
}
</style>
