
<template>
  <a-spin :loading="bit_loading" class="grid-one grid-gap-5">
    <a-typography-text
      >{{ t("按规律生成日期") }}👉
      <a-typography-text type="primary" @click="helpVoid">
        {{ t("查看教程") }}
      </a-typography-text>
    </a-typography-text>
    <a-divider>{{ t("设置日期规律") }}</a-divider>
    <addRulePop @add="addRule" ref="editPop"></addRulePop>
    <a-table
      @change="sortChange"
      :columns="columns"
      :data="dataArr"
      :pagination="false"
      :scroll="{ x: '101%' }"
      :draggable="{ type: 'handle', width: 0 }"
    >
      <template #delete="{ rowIndex }">
        <icon-minus-circle
          size="20"
          style="color: #999"
          @click="deleteVoid(rowIndex)"
        />
      </template>
      <template #edit="{ record, rowIndex }">
        <icon-edit
          size="20"
          style="color: #999"
          @click="selectRowVoid(record, rowIndex)"
        />
      </template>
    </a-table>
    <div class="grid-one border grid-gap-1">
      <div class="row-start-center p-all-5 bg-color">
        <a-typography-text>{{ t('相邻周合并') }}</a-typography-text>
        <a-switch class="m-lr-10" v-model="ty_config.week_merge"></a-switch>
        <a-typography-text
          class="flex-grow"
          :ellipsis="{ rows: 1, showTooltip: true }"
          >{{ t("相邻周合并说明") }}</a-typography-text
        >
      </div>
      <div class="row-start-center p-all-5 bg-color">
        <a-typography-text>{{ t("统一设置") }}</a-typography-text>
        <a-switch class="m-lr-10" v-model="ty_config.is_ty"></a-switch>
        <a-typography-text
          class="flex-grow"
          :ellipsis="{ rows: 1, showTooltip: true }"
          >{{ t("guizeshuoming") }}</a-typography-text
        >
      </div>
      <div class="row-start-center p-all-5" v-if="ty_config.is_ty">
        <a-typography-text class="flex-shrink">{{
          t("总条数")
        }}</a-typography-text>
        <a-input-number
          class="flex-shrink m-lr-5"
          style="flex: 2"
          :min="1"
          v-model="ty_config.totalNum"
          :placeholder="t('总条数')"
        ></a-input-number>

        <a-typography-text class="flex-shrink">{{
          t("开始日期")
        }}</a-typography-text>
        <a-date-picker
          class="flex-shrink m-lr-5"
          style="flex: 2"
          :placeholder="t('开始日期')"
          v-model="ty_config.startDate"
        ></a-date-picker>
        <a-typography-text class="flex-shrink m-r-5">{{
          t("执行模式")
        }}</a-typography-text>
        <a-select
          v-model="ty_config.model"
          style="width: 120px"
          :placeholder="t('选择执行模式')"
          :options="[
            { label: t('强顺序'), value: 1 },
            { label: t('弱顺序'), value: 2 },
          ]"
        >
        </a-select>
        <!-- <a-button type="primary" status="success">确定</a-button> -->
      </div>
    </div>

    <a-divider>{{ t("映射多维表格") }}</a-divider>
    <SelectTableView
      :title="t('选择表')"
      canAdd
      v-model="export_table_id"
      :allFieldDic="{}"
      :preSetArr="[t('选择表')]"
    ></SelectTableView>
    <SelectFieldView
      :title="t('开始日期')"
      canAdd
      :typeNumArr="[5]"
      v-model="export_filed_dic.start_date_filed"
      :allFieldDic="export_filed_dic"
    ></SelectFieldView>
    <SelectFieldView
      v-if="isDateRange"
      :title="t('结束日期')"
      canAdd
      :typeNumArr="[5]"
      v-model="export_filed_dic.end_date_filed"
      :allFieldDic="export_filed_dic"
    ></SelectFieldView>
    <SelectFieldView
      :title="t('周几')"
      v-if="isMustWeek"
      canAdd
      :typeNumArr="[1]"
      v-model="export_filed_dic.week_filed"
      :allFieldDic="export_filed_dic"
    ></SelectFieldView>
    <div class="row-between-center m-t-10">
      <a-typography-text class="font-bold"
        >{{ t("共有数据") }}:{{
          ty_config.is_ty ? ty_config.totalNum : totalNum
        }}</a-typography-text
      >
      <a-space>
        <a-button type="dashed" status="success" @click="importData(true)">{{
          t("预览")
        }}</a-button>
        <a-button type="primary" @click="importData()">{{
          t("导入")
        }}</a-button>
      </a-space>
    </div>
    <!-- 预览 -->
    <a-modal
      :hide-cancel="true"
      :visible="showPrewTable"
      :title="t('日期预览')"
      @close="hidePrewTable"
      @cancel="hidePrewTable"
      width="80%"
      @ok="hidePrewTable"
    >
      <a-table :columns="preColumns" :data="prewArr" :scroll="{y:600}"></a-table>
    </a-modal>
  </a-spin>
</template>

<script setup>
import { ref, onMounted, computed, watch, onUnmounted } from "vue";
import addRulePop from "./addRulePop.vue";
import SelectTableView from "./superView/selectTable.vue";
import SelectFieldView from "./superView/SelectField.vue";
import {
  bit_loading,
  export_table_id,
  addBitRecord,
  export_filed_dic,
  bit_table,
  getTableAllFieldFromId,
} from "./js/superBase";
import dayjs from "dayjs";
import { Message } from "@arco-design/web-vue";
import { cloneDeep } from "lodash";
import { useI18n } from "vue-i18n";
const { t } = useI18n();
const is_tyConfig = ref(false);
const showPrewTable = ref(false);
const ty_config = ref({
  is_ty: false,
  totalNum: 1,
  model: 2, //1强顺序 2弱顺序
  startDate: "",
  week_merge:false//周合并
});
const dataArr = ref([]);
const prewArr = ref([]);
function addRule(item) {
  dataArr.value.push(item);
}
const isMustWeek = computed(() => {
  const findWeekInx = dataArr.value.findIndex(
    (a) => a["configDic"]["selectRuleType"] == "week"
  );
  return findWeekInx >= 0;
});
const isDateRange = computed(() => {
  const findDateRangeInx = dataArr.value.findIndex(
    (a) => a["configDic"]["is_time_range"]
  );
  return findDateRangeInx >= 0;
});
const totalNum = computed(() => {
  let num = 0;
  for (let item of dataArr.value) {
    num = num + item.resultArr.length;
  }
  return num;
});
function hidePrewTable() {
  showPrewTable.value = false;
}

async function importData(isPrew) {
  if (dataArr.value.length == 0) {
    return Message.info(t("请设置时间规律"));
  }
  if (ty_config.value.is_ty) {
    if (ty_config.value.totalNum < 1) {
      return Message.info(t("请输入条数"));
    }
    if (!ty_config.value.startDate) {
      return Message.info(t("请选择开始日期"));
    }
  }

  let arr = [];
  const configArr = dataArr.value.map((a) => a["configDic"]);

  if (ty_config.value.is_ty) {
    prewArr.value = editPop.value.ty_create_from_week(
      configArr,
      ty_config.value
    );
  } else {
    prewArr.value = editPop.value.create_from_week(configArr,ty_config.value);
  }
  if (isPrew) {
    showPrewTable.value = true;
    return;
  }
  if (!export_table_id.value) {
    return Message.info(t("选择映射的表"));
  }

  if (!export_filed_dic.value.start_date_filed) {
    return Message.info(t("选择开始日期"));
  }
  if (!export_filed_dic.value.end_date_filed && isDateRange.value) {
    return Message.info(t("选择结束日期"));
  }
  if (!export_filed_dic.value.week_filed && isMustWeek.value) {
    return Message.info(t("选择周几"));
  }

  for (let item of prewArr.value) {
    let fields = {};

    fields[export_filed_dic.value["start_date_filed"]] = dayjs(
      item.times[0]
    ).valueOf();
    if (item.times.length == 2) {
      fields[export_filed_dic.value["end_date_filed"]] = dayjs(
        item.times[1]
      ).valueOf();
    }
    fields[export_filed_dic.value["week_filed"]] = item.week.label
    arr.push({ fields });
  }

  console.log("dddd", arr);
  await addBitRecord(arr, export_table_id.value);
  localStorage.setItem("SSDATECONFIG", dataArr.value);
  Message.success(t("导入成功"));

  // const dd = await getTableAllFieldFromId(export_table_id.value);
  // const view = await bit_table.getActiveView();
  // console.log("recordIdList", recordIdList);
}
// 选中的行
const editPop = ref();
function selectRowVoid(record, rowIndex) {
  editPop.value.showPopVoid(true, {
    configDic: cloneDeep(record.configDic),
    success: (newItem) => {
      dataArr.value[rowIndex] = newItem;
    },
  });
}
function sortChange(data) {
  dataArr.value = data;
}
// 删除
function deleteVoid(inx) {
  dataArr.value.splice(inx, 1);
}
const columns = ref([]);
const preColumns = ref([]);
onMounted(() => {
  preColumns.value = [
    {
      title: t("开始时间"),
      dataIndex: "times",
      align: "center",
      ellipsis: true,
      tooltip: true,
      render: ({ record }) => record["times"][0],
    },
    {
      title: t("结束时间"),
      dataIndex: "table.ruleName",
      render: ({ record }) =>
        record["times"].length > 1 ? record["times"][1] : "",
      ellipsis: true,
      align: "center",
      tooltip: true,
    },
    {
      title: t("星期"),
      align: "center",
      dataIndex: "week.label"
      // render: ({ record }) =>record['week']
    },
  ];
  columns.value = [
    {
      title: t("删除"),
      dataIndex: "delete",
      slotName: "delete",
      width: 60,
    },
    {
      title: t("规律"),
      dataIndex: "table.ruleName",
      ellipsis: true,
      tooltip: true,
      width: 60,
      headerCellClass: "thTitle",
    },
    {
      title: t("日期范围"),
      dataIndex: "table.dateRange",
      ellipsis: true,
      tooltip: true,
      cellClass: "thTitle",
      render: ({ record }) => {
        if (ty_config.value.is_ty) {
          return "";
        }
        return record["table"]["dateRange"];
      },
    },
    {
      title: t("时间"),
      dataIndex: "table.times",
      ellipsis: true,
      tooltip: true,
    },
    {
      title: t("条数"),
      dataIndex: "table.maxNum",
      width: 60,
      ellipsis: true,
      tooltip: true,
      render: ({ record }) => {
        if (ty_config.value.is_ty) {
          return 1;
        }
        if (record.configDic.createType == "input_maxNum") {
          return record.table.maxNum;
        } else {
          return record.resultArr.length;
        }
      },
    },
    {
      title: t("编辑"),
      dataIndex: "edit",
      slotName: "edit",
      width: 60,
      fixed: "right",
    },
  ];
});

function helpVoid() {
  window.open(
    "https://y35xdslz6g.feishu.cn/docx/Yx0Sd6IRboXYroxG7fic5beDnid?from=from_copylink",
    "_blank"
  );
}
</script>

<style>
.labelCss {
  width: 70px;
  text-align: center;
  flex-shrink: 0;
}
.arco-typography {
  margin-bottom: 0px !important;
}
.thTitle {
  color: red;
  font-size: 20px;
}
</style>
