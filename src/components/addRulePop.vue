
<template>
  <div>
    <div class="row-between-center m-b-10">
      <a-typography-text class="font-bold" style="font-size: 16px"
        >设置规律</a-typography-text
      >
      <a-button type="primary" @click="showPopVoid(true)">新增规律</a-button>
    </div>

    <a-modal
      title="设置日期规律"
      :visible="showPop"
      width="400px"
      @before-ok="commitVoid"
      @cancel="showPopVoid(false)"
      @close="showPopVoid(false)"
    >
      <div class="grid-one m-t-10">
        <div class="row-start-center">
          <a-typography-text class="labelClass">选择规律</a-typography-text>
          <a-radio-group v-model="configDic.selectRuleType">
            <a-radio value="week">周规律</a-radio>
            <a-radio value="day">日规律</a-radio>
          </a-radio-group>
        </div>
        <!-- 按照周的规则 -->
        <div v-if="configDic.selectRuleType == 'week'" class="row-start-center">
          <a-typography-text class="labelClass">选择周</a-typography-text>
          <a-select
            :options="optionsWeek"
            multiple
            v-model="configDic.input_selectWeek"
            placeholder="选择周几"
          ></a-select>
        </div>
        <!-- 日期按照相隔 -->
        <div v-if="configDic.selectRuleType == 'day'" class="row-start-center">
          <a-typography-text class="labelClass">间隔天数</a-typography-text>
          <a-input-number
            mode="button"
            size="large"
            v-model="configDic.input_spaceDayNum"
            :min="1"
          ></a-input-number>
        </div>
        <div class="row-start-center">
          <a-typography-text class="labelClass">时间点</a-typography-text>
          <a-radio-group v-model="configDic.is_time_range">
            <a-radio :value="false">时间点</a-radio>
            <a-radio :value="true">时间段</a-radio>
          </a-radio-group>
        </div>
        <div class="row-start-center">
          <a-typography-text class="labelClass">时间</a-typography-text>
          <a-time-picker
            v-if="configDic.is_time_range"
            type="time-range"
            class="flex-grow"
            format="HH:mm"
            v-model="configDic.input_time_range"
          />
          <a-time-picker
            v-else
            class="flex-grow"
            v-model="configDic.input_time"
            format="HH:mm"
          />
        </div>
        <div class="row-start-center">
          <a-typography-text class="labelClass">限制方式</a-typography-text>
          <a-radio-group v-model="configDic.createType">
            <a-radio value="input_maxNum">条数限制</a-radio>
            <a-radio value="input_dateRange">日期范围</a-radio>
          </a-radio-group>
        </div>
        <!-- 生成条数 -->
        <div
          v-if="configDic.createType == 'input_maxNum'"
          class="row-start-center"
        >
          <a-typography-text class="labelClass">天数</a-typography-text>
          <a-input-number
            mode="button"
            size="large"
            v-model="configDic.input_maxNum"
            :min="1"
          ></a-input-number>
        </div>
        <div
          v-if="configDic.createType == 'input_maxNum'"
          class="row-start-center"
        >
          <a-typography-text class="labelClass">开始日期</a-typography-text>
          <a-date-picker
            v-model="configDic.input_start_date"
            class="flex-grow"
          ></a-date-picker>
        </div>
        <!-- 日期范围 -->
        <div
          v-if="configDic.createType == 'input_dateRange'"
          class="row-start-center"
        >
          <a-typography-text class="labelClass">日期范围</a-typography-text>
          <a-range-picker
            v-model="configDic.input_dateRange"
            class="flex-grow"
          />
        </div>
        <!-- <a-button @click="sureVoid">确定</a-button> -->
      </div>
    </a-modal>
  </div>
</template>
<script setup>
import { cloneDeep } from "lodash";
import { computed, ref, watch, watchEffect } from "vue";
import { Message } from "@arco-design/web-vue";
import dayjs from "dayjs";
import weekday from "dayjs/plugin/weekday";
import "dayjs/locale";
dayjs.locale("zh-cn");

// var weekday = require('dayjs/plugin/weekday')
dayjs.extend(weekday);
const configDic = ref({
  selectRuleType: "week",
  input_selectWeek: [], //选中的周
  input_spaceDayNum: 1, //间隔天数
  is_time_range: true, //false 时间端  true
  input_time: "", //时间点
  input_time_range: ["08:00", "08:30"], //时间点
  createType: "input_maxNum",
  input_maxNum: 10, //最大条数
  input_start_date: "", //日期
  input_dateRange: [], //日期范围
});

const optionsWeek = ref([
  { label: "周一", value: 1 },
  { label: "周二", value: 2 },
  { label: "周三", value: 3 },
  { label: "周四", value: 4 },
  { label: "周五", value: 5 },
  { label: "周六", value: 6 },
  { label: "周日", value: 0 },
]);
defineExpose({ showPopVoid, ty_create_from_week, create_from_week });
let editItemDic = { configDic: "", success: () => {} };
const showPop = ref(false);
function showPopVoid(show, back) {
  showPop.value = show;
  if (back) {
    editItemDic = back;
    configDic.value = back.configDic;
  } else {
    editItemDic = "";
  }
}
const emit = defineEmits(["add"]);
function commitVoid(done) {
  let dic = {
    ruleName: "",
    times: "",
    dateRange: "",
    maxNum: "",
  };
  if (configDic.value.selectRuleType == "week") {
    if (configDic.value.input_selectWeek.length == 0) {
      done(false);
      return Message.info("请选择周");
    }
    const weeks = optionsWeek.value
      .filter((a) => configDic.value.input_selectWeek.includes(a["value"]))
      .map((a) => a["label"]);
    dic.ruleName = weeks.join(",");
  } else {
    dic.ruleName = "间隔" + configDic.value.input_maxNum + "天";
  }

  if (configDic.value.is_time_range) {
    if (configDic.value.input_time_range.length == 2) {
      dic.times = configDic.value.input_time_range.join("~");
    } else {
      done(false);
      return Message.info("请选择时间");
    }
  } else {
    if (!configDic.value.input_time) {
      done(false);
      return Message.info("请选择时间");
    } else {
      dic.times = [configDic.value.input_time];
    }
  }

  if (configDic.value.createType == "input_maxNum") {
    dic.maxNum = configDic.value.input_maxNum;
    if (!configDic.value.input_start_date) {
      done(false);
      return Message.info("请选择开始日期");
    } else {
      dic.dateRange = configDic.value.input_start_date;
    }
  }
  if (configDic.value.createType == "input_dateRange") {
    if (configDic.value.input_dateRange.length == 2) {
      dic.dateRange = [configDic.value.input_dateRange].join("~");
    } else {
      done(false);
      return Message.info("请选择日期范围");
    }
  }
  // 生成描述

  const weekStr = optionsWeek.value
    .filter((a) => configDic.value.input_selectWeek.includes(a["value"]))
    .map((a) => a["label"]);
  const resultDic = {
    resultArr: create_from_week([configDic.value]),
    table: dic,
    configDic: cloneDeep(configDic.value),
  };
  if (editItemDic.configDic) {
    editItemDic.success(resultDic);
  } else {
    emit("add", resultDic);
  }

  done(true);
  showPopVoid(false);
}

function sureVoid() {
  create_from_week(configDic.value);
}
function create_from_week(configArr) {
  let arr = [];

  for (let dic of configArr) {
    const startDate =
      dic.createType == "input_maxNum"
        ? dic.input_start_date
        : dic.input_dateRange[0];
    const endDate =
      dic.createType == "input_maxNum" ? "" : dic.input_dateRange[1];
    // 限制条数
    let startTime = new Date(startDate).getTime();
    const endTime = new Date(endDate).getTime();
    // 循环条件
    function loopCondition(num) {
      if (dic.createType == "input_maxNum") {
        return arr.length < dic.input_maxNum;
      }
      if (dic.createType == "input_dateRange") {
        const curTime = dayjs(startDate).add(num, "d").valueOf();
        return curTime <= endTime;
      }
    }
    let num = 0;
    while (loopCondition(num)) {
      const dicc = { times: [], week: "" };
      const curDate = dayjs(startDate).add(num, "d").format("YYYY-MM-DD");
      const weekNum = dayjs(curDate).day();
      if (dic.selectRuleType == "week") {
        if (dic.input_selectWeek.includes(weekNum)) {
          if (dic.is_time_range) {
            dicc.times.push(curDate + " " + dic.input_time_range[0]);
            dicc.times.push(curDate + " " + dic.input_time_range[1]);
          } else {
            dicc.times.push(curDate + " " + dic.input_time);
          }
          const weekItem = optionsWeek.value.find((a) => a["value"] == weekNum);
          dicc.week = weekItem.label;
          arr.push(dicc);
        }
      } else {
        if (dic.is_time_range) {
          dicc.times.push(curDate + " " + dic.input_time_range[0]);
          dicc.times.push(curDate + " " + dic.input_time_range[1]);
        } else {
          dicc.times.push(curDate + " " + dic.input_time);
        }
        const weekItem = optionsWeek.value.find((a) => a["value"] == weekNum);
        dicc.week = weekItem.label;
        arr.push(dicc);
      }
      num = num + (dic.selectRuleType == "week" ? 1 : dic.input_spaceDayNum);
    }
  }
  return arr;
}

// 统一创建
function ty_create_from_week(configArr, ty_config) {
  // 限制条数
  let arr = [];
  const startDate = ty_config.startDate;

  // 循环条件
  function loopCondition(num) {
    return arr.length < ty_config.totalNum;
  }
  let num = 0;
  let curDate = "";
  let weekNum = "";
  while (loopCondition(num)) {
    debugger;
    for (let dic of configArr) {
      const dicc = { times: [], week: "" };
      // 如果是周  则循环 直到找齐所有的周 [周一,周二,周三]知道找齐所有的周一 周二 周三
      if (dic.selectRuleType == "week") {
        let i = 0;
        while (i < dic.input_selectWeek.length) {
          curDate = dayjs(startDate).add(num, "d").format("YYYY-MM-DD");
          weekNum = dayjs(curDate).day();
          num++;
          if (dic.input_selectWeek.includes(weekNum)) {
            if (dic.is_time_range) {
              dicc.times.push(curDate + " " + dic.input_time_range[0]);
              dicc.times.push(curDate + " " + dic.input_time_range[1]);
            } else {
              dicc.times.push(curDate + " " + dic.input_time);
            }
            const weekItem = optionsWeek.value.find(
              (a) => a["value"] == weekNum
            );
            dicc.week = weekItem.label;
            i++;
            arr.push(dicc);
            break;
          }
        }
      } else {
        curDate = dayjs(startDate).add(num, "d").format("YYYY-MM-DD");
        weekNum = dayjs(curDate).day();
        // 如果是日规律 则循环一次
        if (dic.is_time_range) {
          dicc.times.push(curDate + " " + dic.input_time_range[0]);
          dicc.times.push(curDate + " " + dic.input_time_range[1]);
        } else {
          dicc.times.push(curDate + " " + dic.input_time);
        }
        const weekItem = optionsWeek.value.find((a) => a["value"] == weekNum);
        dicc.week = weekItem.label;
        arr.push(dicc);
        num = num + dic.input_spaceDayNum;
      }
    }
  }

  return arr;
}
</script>

<style>
.labelClass {
  width: 80px;
  flex-shrink: 0;
}
</style>
