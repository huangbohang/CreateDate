
<template>
  <div>
    <div class="row-between-center m-b-10">
      <a-typography-text
        class="font-bold flex-shrink"
        style="font-size: 16px"
        >{{ t("设置规律") }}</a-typography-text
      >
      <!-- <a-select :options="[{ label: 'ddd', value: 1 }]">
        <template #option="{ data }">
          <div class="row-start-center">
            <a-input v-model="data.label"></a-input>
            <icon-edit />
          </div>
        </template>
      </a-select> -->
      <a-button type="primary" @click="showPopVoid(true)">{{
        t("新增规律")
      }}</a-button>
    </div>

    <a-modal
      :title="t('设置日期规律')"
      :visible="showPop"
      width="380px"
      @before-ok="commitVoid"
      @cancel="showPopVoid(false)"
      @close="showPopVoid(false)"
    >
      <div class="grid-one m-t-10">
        <div class="row-start-center">
          <a-typography-text class="labelClass">{{
            t("选择规律")
          }}</a-typography-text>
          <a-radio-group v-model="configDic.selectRuleType">
            <a-radio value="week">{{ t("周规律") }}</a-radio>
            <a-radio value="day">{{ t("日规律") }}</a-radio>
          </a-radio-group>
        </div>
        <!-- 按照周的规则 -->
        <div v-if="configDic.selectRuleType == 'week'" class="row-start-center">
          <a-typography-text class="labelClass">{{
            t("选择周")
          }}</a-typography-text>
          <a-select
            :options="optionsWeek"
            multiple
            v-model="configDic.input_selectWeek"
            :placeholder="t('选择周几')"
          ></a-select>
        </div>
        <!-- 日期按照相隔 -->
        <div v-if="configDic.selectRuleType == 'day'" class="row-start-center">
          <a-typography-text class="labelClass">{{
            t("间隔天数")
          }}</a-typography-text>
          <a-input-number
            mode="button"
            size="large"
            v-model="configDic.input_spaceDayNum"
            :min="1"
          ></a-input-number>
        </div>
        <div class="row-start-center">
          <a-typography-text class="labelClass">{{
            t("时间点")
          }}</a-typography-text>
          <a-radio-group v-model="configDic.is_time_range">
            <a-radio :value="false">{{ t("时间点") }}</a-radio>
            <a-radio :value="true">{{ t("时间段") }}</a-radio>
          </a-radio-group>
        </div>
        <div class="row-start-center">
          <a-typography-text class="labelClass">{{
            t("时间")
          }}</a-typography-text>
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
          <a-typography-text class="labelClass">{{
            t("限制方式")
          }}</a-typography-text>
          <a-radio-group v-model="configDic.createType">
            <a-radio value="input_maxNum">{{ t("条数限制") }}</a-radio>
            <a-radio value="input_dateRange">{{ t("日期范围") }}</a-radio>
          </a-radio-group>
        </div>
        <!-- 生成条数 -->
        <div
          v-if="configDic.createType == 'input_maxNum'"
          class="row-start-center"
        >
          <a-typography-text class="labelClass">{{
            t("天数")
          }}</a-typography-text>
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
          <a-typography-text class="labelClass">{{
            t("开始日期")
          }}</a-typography-text>
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
          <a-typography-text class="labelClass">{{
            t("日期范围")
          }}</a-typography-text>
          <a-range-picker
            v-model="configDic.input_dateRange"
            class="flex-grow"
          />
        </div>
        <!-- <a-button @click="sureVoid">{{t('确定')}}</a-button> -->
      </div>
    </a-modal>
  </div>
</template>
<script setup>
import { cloneDeep } from "lodash";
import { computed, onMounted, ref, watch, watchEffect } from "vue";
import { Message } from "@arco-design/web-vue";
import dayjs from "dayjs";
import weekday from "dayjs/plugin/weekday";
import "dayjs/locale";
dayjs.locale("zh-cn");
import { useI18n } from "vue-i18n";
const { t } = useI18n();
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
  id: 0,
});
const optionsWeek = ref([]);
onMounted(() => {
  optionsWeek.value = [
    { label: t("周一"), value: 1 },
    { label: t("周二"), value: 2 },
    { label: t("周三"), value: 3 },
    { label: t("周四"), value: 4 },
    { label: t("周五"), value: 5 },
    { label: t("周六"), value: 6 },
    { label: t("周日"), value: 0 },
  ];
});

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
    configDic.value.id = new Date().getTime();
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
      return Message.info(t("请选择周"));
    }
    const weeks = optionsWeek.value
      .filter((a) => configDic.value.input_selectWeek.includes(a["value"]))
      .map((a) => a["label"]);
    dic.ruleName = weeks.join(",");
  } else {
    dic.ruleName = t("间隔") + configDic.value.input_maxNum + t("天");
  }

  if (configDic.value.is_time_range) {
    if (configDic.value.input_time_range.length == 2) {
      dic.times = configDic.value.input_time_range.join("~");
    } else {
      done(false);
      return Message.info(t("请选择时间"));
    }
  } else {
    if (!configDic.value.input_time) {
      done(false);
      return Message.info(t("请选择时间"));
    } else {
      dic.times = [configDic.value.input_time];
    }
  }

  if (configDic.value.createType == "input_maxNum") {
    dic.maxNum = configDic.value.input_maxNum;
    if (!configDic.value.input_start_date) {
      done(false);
      return Message.info(t("请选择开始日期"));
    } else {
      dic.dateRange = configDic.value.input_start_date;
    }
  }
  if (configDic.value.createType == "input_dateRange") {
    if (configDic.value.input_dateRange.length == 2) {
      dic.dateRange = [configDic.value.input_dateRange].join("~");
    } else {
      done(false);
      return Message.info(t("请选择日期范围"));
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
function create_from_week(configArr, ty_config = { week_merge: false }) {
  let arr = [];
  let week_merge_date = {};

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
    let createNum = 0;
    // 循环条件
    function loopCondition(num) {
      if (dic.createType == "input_maxNum") {
        return createNum < dic.input_maxNum;
      }
      if (dic.createType == "input_dateRange") {
        const curTime = dayjs(startDate).add(num, "d").valueOf();
        return curTime <= endTime;
      }
    }
    let num = 0;
    while (loopCondition(num)) {
      const dicc = { times: [], week: "" };
      let curDate = dayjs(startDate).add(num, "d").format("YYYY-MM-DD");
      const weekNum = dayjs(curDate).day();
      if (dic.selectRuleType == "week") {
        if (dic.input_selectWeek.includes(weekNum)) {
          // 周合并开启后,相同周以第一个符合的为准
          if (ty_config.week_merge) {
            if (week_merge_date[weekNum]) {
              curDate = week_merge_date[weekNum];
            } else {
              week_merge_date[weekNum] = curDate;
            }
          }

          if (dic.is_time_range) {
            dicc.times.push(curDate + " " + dic.input_time_range[0]);
            dicc.times.push(curDate + " " + dic.input_time_range[1]);
          } else {
            dicc.times.push(curDate + " " + dic.input_time);
          }
          const weekItem = optionsWeek.value.find((a) => a["value"] == weekNum);
          dicc.week = weekItem.label;
          createNum++;
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
        createNum++;
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
  let week_merge_date = {};
  const startDate = ty_config.startDate;

  // 循环条件
  function loopCondition(num) {
    return arr.length < ty_config.totalNum;
  }
  let num = 0;
  let curDate = "";
  let weekNum = "";
  let all_input_selectWeek = [];
  if (ty_config.model == 2) {
    for (let item of configArr) {
      if (item.selectRuleType == "week") {
        all_input_selectWeek.push(item);
      }
    }
  } else {
  }
  while (loopCondition()) {
    week_merge_date = {};
    for (let dic of configArr) {
      // 如果是周  则循环 直到找齐所有的周 [周一,周二,周三]知道找齐所有的周一 周二 周三
      if (dic.selectRuleType == "week") {
        let i = 0;
        while (i < dic.input_selectWeek.length) {
          curDate = dayjs(startDate).add(num, "d").format("YYYY-MM-DD");
          weekNum = dayjs(curDate).day();
          num++;
          if (ty_config.model == 1) {
            all_input_selectWeek = [dic];
          }
          // 找到对应的周

          const findItem = all_input_selectWeek.find((a) =>
            a.input_selectWeek.includes(weekNum)
          );
          let lastDic = "";
          if (findItem) {
            // 最后一个
            if (arr.length > 0) {
              lastDic = arr[arr.length - 1];
              const downWeekNum =
                parseInt(lastDic.week.value) + 1 == 7
                  ? 0
                  : parseInt(lastDic.week.value) + 1;
              if (downWeekNum == weekNum) {
                curDate = dayjs(lastDic["times"][0])
                  .add(1, "d")
                  .format("YYYY-MM-DD");
                week_merge_date = {};
                // if (ty_config.model == 1) {
                num = lastDic.num;
                // }
              }
            }
            // 周合并开启后,相同周以第一个符合的为准
            if (ty_config.week_merge) {
              if (week_merge_date[weekNum]) {
                if (ty_config.model == 1) {
                  const lastWeekNum = lastDic.week.value;
                  if (lastWeekNum == weekNum) {
                    curDate = week_merge_date[weekNum]["date"];
                    num = lastDic.num;
                  }
                } else {
                  curDate = week_merge_date[weekNum]["date"];
                }
              } else {
                week_merge_date[weekNum] = { date: curDate, weekNum: weekNum };
              }
            }
            const dicc = { times: [], week: "", num: num };
            if (dic.is_time_range) {
              dicc.times.push(curDate + " " + findItem.input_time_range[0]);
              dicc.times.push(curDate + " " + findItem.input_time_range[1]);
            } else {
              dicc.times.push(curDate + " " + findItem.input_time);
            }
            const weekItem = optionsWeek.value.find(
              (a) => a["value"] == weekNum
            );
            dicc.week = weekItem;
            i++;
            if (loopCondition()) {
              findItem["sort"] = num;
              // 排序解决同样周的问题
              all_input_selectWeek = all_input_selectWeek.sort((a, b) => {
                if (a - b > 0) {
                  return 1;
                } else {
                  return -1;
                }
              });
              arr.push(dicc);
            }
          }
        }
        // 找到已经处理过的
        // const findIndex = all_input_selectWeek.find((a) => a.id == dic.id);
        // all_input_selectWeek.splice(findIndex, 1);
      } else {
        curDate = dayjs(startDate).add(num, "d").format("YYYY-MM-DD");
        weekNum = dayjs(curDate).day();
        const dicc = { times: [], week: "" };
        // 如果是日规律 则循环一次
        if (dic.is_time_range) {
          dicc.times.push(curDate + " " + dic.input_time_range[0]);
          dicc.times.push(curDate + " " + dic.input_time_range[1]);
        } else {
          dicc.times.push(curDate + " " + dic.input_time);
        }
        const weekItem = optionsWeek.value.find((a) => a["value"] == weekNum);
        dicc.week = weekItem.label;
        if (loopCondition()) {
          arr.push(dicc);
        }
        num = num + dic.input_spaceDayNum;
      }
      //
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
