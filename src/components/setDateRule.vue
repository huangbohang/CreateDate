
<template>
  <a-spin :loading="bit_loading" class="grid-one grid-gap-5">
    <a-typography-text
      >按规律生成日期👉
      <a-typography-text type="primary" @click="helpVoid">
        查看教程
      </a-typography-text>
    </a-typography-text>
    <a-divider>设置日期规律</a-divider>
    <addRulePop @add="addRule" ref="editPop"></addRulePop>
    <a-table
      @change="sortChange"
      :columns="columns"
      :data="dataArr"
      :pagination="false"
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
        <a-typography-text>统一设置</a-typography-text>
        <a-switch class="m-lr-10" v-model="ty_config.is_ty"></a-switch>
        <a-typography-text
          class="flex-grow"
          :ellipsis="{ rows: 1, showTooltip: true }"
          >开启后,以上规律天数默认为1,日期范围失效,第一个规则以统一开始日期为准,第二个规则开始时间为第一个规则满足条件后第二天开始,依次类推</a-typography-text
        >
      </div>
      <div class="row-start-center p-all-5" v-if="ty_config.is_ty">
        <a-typography-text class="flex-shrink">总条数</a-typography-text>
        <a-input-number
          class="flex-shrink m-lr-5"
          style="flex: 2"
          :min="1"
          v-model="ty_config.totalNum"
          placeholder="总条数"
        ></a-input-number>

        <a-typography-text class="flex-shrink">开始时间</a-typography-text>
        <a-date-picker
          class="flex-shrink m-lr-5"
          style="flex: 2"
          placeholder="开始日期"
          v-model="ty_config.startDate"
        ></a-date-picker>
        <!-- <a-button type="primary" status="success">确定</a-button> -->
      </div>
    </div>

    <a-divider>映射多维表格</a-divider>
    <SelectTableView
      title="选择表"
      canAdd
      v-model="export_table_id"
      :allFieldDic="{ dy_user_table_id }"
      :preSetArr="['抖音视频表']"
    ></SelectTableView>
    <SelectFieldView
      title="开始日期"
      canAdd
      :typeNumArr="[5]"
      v-model="export_filed_dic.start_date_filed"
      :allFieldDic="export_filed_dic"
    ></SelectFieldView>
    <SelectFieldView
      v-if="isDateRange"
      title="结束日期"
      canAdd
      :typeNumArr="[5]"
      v-model="export_filed_dic.end_date_filed"
      :allFieldDic="export_filed_dic"
    ></SelectFieldView>
    <SelectFieldView
      title="周几"
      v-if="isMustWeek"
      canAdd
      :typeNumArr="[1]"
      v-model="export_filed_dic.week_filed"
      :allFieldDic="export_filed_dic"
    ></SelectFieldView>
    <div class="row-between-center m-t-10">
      <a-typography-text class="font-bold"
        >共有数据:{{ ty_config.is_ty?ty_config.totalNum:totalNum }}</a-typography-text
      >
      <a-space>
        <a-button type="dashed" status="success" @click="importData(true)"
          >预览</a-button
        >
        <a-button type="primary" @click="importData()">导入</a-button>
      </a-space>
    </div>
    <!-- 预览 -->
    <a-modal
    :hide-cancel="true"
      :visible="showPrewTable"
      title="日期预览"
      @close="hidePrewTable"
      @cancel="hidePrewTable"
      @ok="hidePrewTable"
    >
      <a-table :columns="preColumns" :data="prewArr"></a-table>
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
const is_tyConfig = ref(false);
const showPrewTable = ref(false);
const ty_config = ref({
  is_ty: false,
  totalNum: 1,
  startDate: "",
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
    return Message.info("请设置时间规律");
  }
  if (ty_config.value.is_ty) {
    if (ty_config.value.totalNum < 1) {
      return Message.info("请输入条数");
    }
    if (!ty_config.value.startDate) {
      return Message.info("请选择开始日期");
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
    prewArr.value = editPop.value.create_from_week(configArr);
  }
  if (isPrew) {
    showPrewTable.value = true;

    return;
  }
  if (!export_table_id.value) {
    return Message.info("选择映射的表");
  }

  if (!export_filed_dic.value.start_date_filed) {
    return Message.info("选择开始日期");
  }
  if (!export_filed_dic.value.end_date_filed && isDateRange.value) {
    return Message.info("选择结束日期");
  }
  if (!export_filed_dic.value.week_filed && isMustWeek.value) {
    return Message.info("选择周几");
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
    fields[export_filed_dic.value["week_filed"]] = item.week;
    arr.push({ fields });
  }

  await addBitRecord(arr, export_table_id.value);
  localStorage.setItem("SSDATECONFIG", dataArr.value);
  Message.success("导入成功");

  // const dd = await getTableAllFieldFromId(export_table_id.value);
  // const view = await bit_table.getActiveView();
  // const recordIdList = await view.getVisibleRecordIdList();
  // console.log("recordIdList", recordIdList);
}
// 选中的行
const editPop = ref();
function selectRowVoid(record, rowIndex) {
  editPop.value.showPopVoid(true, {
    configDic: cloneDeep(record.configDic),
    success: (newItem) => {
      dataArr.value[rowIndex] = newItem;
      console.log("成功", newItem);
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

const columns = ref([
  {
    title: "删除",
    dataIndex: "delete",
    slotName: "delete",
    width: "60",
  },
  {
    title: "规律",
    dataIndex: "table.ruleName",
    ellipsis: true,
    tooltip: true,
  },
  {
    title: "日期范围",
    dataIndex: "table.dateRange",
    ellipsis: true,
    tooltip: true,
    render: ({ record }) => {
      if (ty_config.value.is_ty) {
        return "";
      }
      return record["table"]["dateRange"];
    },
  },
  {
    title: "时间",
    dataIndex: "table.times",
    ellipsis: true,
    tooltip: true,
  },
  {
    title: "条数",
    dataIndex: "table.maxNum",
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
    title: "编辑",
    dataIndex: "edit",
    slotName: "edit",
    width: "60",
  },
]);

function helpVoid() {
  window.open(
    "https://y35xdslz6g.feishu.cn/docx/Yx0Sd6IRboXYroxG7fic5beDnid?from=from_copylink",
    "_blank"
  );
}
const preColumns = ref([
  {
    title: "开始时间",
    dataIndex: "times",
    align: "center",
    render: ({ record }) => record["times"][0],
  },
  {
    title: "结束时间",
    dataIndex: "table.ruleName",
    render: ({ record }) =>
      record["times"].length > 1 ? record["times"][1] : "",
    ellipsis: true,
    align: "center",

    tooltip: true,
  },
  {
    title: "星期",
    align: "center",
    dataIndex: "week",
  },
]);
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
</style>
