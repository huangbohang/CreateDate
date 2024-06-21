<template>
  <div>
    <p @click="createExcel">测111试</p>
  </div>
</template>
<script setup>
import { onMounted, ref } from "vue";
import ExcelJS from "exceljs";
import dayjs from "dayjs";
const dataArr = ref([
  { name: "臧三", sex: "男", age: 100 },
  { name: "类似", sex: "女", age: 10 },
]);
const html = ref("");
onMounted(() => {});

async function createExcel() {
  // Add column headers and define column keys and widths
  // Note: these column structures are a workbook-building convenience only,
  // apart from the column width, they will not be fully persisted.
  const workbook = new ExcelJS.Workbook();
  const worksheet = workbook.addWorksheet("My Sheet");
  if (1 > 2) {
    let fixColumns = [
      { header: "序号", key: "name", width: 10 },
      { header: "发车日期", key: "cheCompany", width: 10 },
      { header: "客户", key: "code", width: 10 },
      { header: "线路", key: "chepaihao", width: 10 },
      { header: "运输类型", key: "xl", width: 10 },
      { header: "司机提成", key: "xl", width: 10 },
      { header: "车牌", key: "xl", width: 10 },
      { header: "运费金额", key: "xl", width: 10 },
    ];
    // 车辆现金支付明细
    let carMoneyArr = [
      { header: "油费", key: "name", width: 10 },
      { header: "过路费", key: "cheCompany", width: 10 },
      { header: "停车费", key: "code", width: 10 },
      { header: "信息费", key: "chepaihao", width: 10 },
      { header: "违章", key: "xl", width: 10 },
      { header: "杂费", key: "xl", width: 10 },
    ];
    // 车头支出
    let carHeadArr = [
      { header: "维修费", key: "name", width: 10 },
      { header: "轮胎", key: "cheCompany", width: 10 },
      { header: "杂费", key: "code", width: 10 },
    ];
    // 车挂支出
    let carEndArr = [
      { header: "维修费", key: "name", width: 10 },
      { header: "轮胎", key: "cheCompany", width: 10 },
      { header: "杂费", key: "code", width: 10 },
    ];
    // 费现金支付明细
    let carNoMoneyArr = [
      { header: "油费", key: "name", width: 10 },
      { header: "油费", key: "name", width: 10 },
      { header: "油费", key: "name", width: 10 },
      { header: "油费", key: "name", width: 10 },
      { header: "过路费", key: "cheCompany", width: 10 },
      { header: "过路费", key: "cheCompany", width: 10 },
      { header: "过路费", key: "cheCompany", width: 10 },
      { header: "过路费", key: "cheCompany", width: 10 },
      { header: "其他", key: "code", width: 10 },
      { header: "非现金小计", key: "chepaihao", width: 10 },
    ];

    worksheet.columns = [
      ...fixColumns,
      ...carMoneyArr,
      ...carHeadArr,
      ...carEndArr,
      ...carNoMoneyArr,
    ];

    // 插入标题行
    const tittleArr = [
      "固定列",
      "车辆现金支付明细",
      "车头支出",
      "车挂支出",
      "非现金支出明细",
    ];

    worksheet.spliceRows(1, 0, tittleArr);

    // 计算需要合并的列数
    let mergeLengths = [
      fixColumns.length,
      carMoneyArr.length,
      carHeadArr.length,
      carEndArr.length,
      carNoMoneyArr.length,
    ];

    // 合并单元格
    let startCol = 1; // 起始列
    for (let i = 0; i < mergeLengths.length; i++) {
      let mergeLength = mergeLengths[i];
      if (mergeLength > 0) {
        worksheet.mergeCells(1, startCol, 1, startCol + mergeLength - 1);
        const titleCell = worksheet.getCell(1, startCol);
        titleCell.value = tittleArr[i];
        // 设置单元格样式，使标题居中
        titleCell.alignment = { vertical: "middle", horizontal: "center" };
        // 设置单元格样式（字体加粗、字号调整等）
        titleCell.font = { size: 300, bold: true };

        startCol += mergeLength;
      }
    }
  } else {
    // 定义每个部分的列和标题
    const sections = [
      {
        title: "固定列",
        columns: [
          { header: "序号", key: "name", width: 10 },
          { header: "发车日期", key: "cheCompany", width: 10 },
          { header: "客户", key: "code", width: 10 },
          { header: "线路", key: "chepaihao", width: 10 },
          { header: "运输类型", key: "xl", width: 10 },
          { header: "司机提成", key: "xl", width: 10 },
          { header: "车牌", key: "xl", width: 10 },
          { header: "运费金额", key: "xl", width: 10 },
        ],
      },
      {
        title: "车辆现金支付明细",
        columns: [
          { header: "油费", key: "name", width: 10 },
          { header: "过路费", key: "cheCompany", width: 10 },
          { header: "停车费", key: "code", width: 10 },
          { header: "信息费", key: "chepaihao", width: 10 },
          { header: "违章", key: "xl", width: 10 },
          { header: "杂费", key: "xl", width: 10 },
        ],
      },
      {
        title: "车头支出",
        columns: [
          { header: "维修费", key: "name", width: 10 },
          { header: "轮胎", key: "cheCompany", width: 10 },
          { header: "杂费", key: "code", width: 10 },
        ],
      },
      {
        title: "车挂支出",
        columns: [
          { header: "维修费", key: "name", width: 10 },
          { header: "轮胎", key: "cheCompany", width: 10 },
          { header: "杂费", key: "code", width: 10 },
        ],
      },
      {
        title: "非现金支出明细",
        columns: [
          { header: "油费", key: "name", width: 10 },
          { header: "油费", key: "name", width: 10 },
          { header: "油费", key: "name", width: 10 },
          { header: "油费", key: "name", width: 10 },
          { header: "过路费", key: "cheCompany", width: 10 },
          { header: "过路费", key: "cheCompany", width: 10 },
          { header: "过路费", key: "cheCompany", width: 10 },
          { header: "过路费", key: "cheCompany", width: 10 },
          { header: "其他", key: "code", width: 10 },
          { header: "非现金小计", key: "chepaihao", width: 10 },
        ],
      },
    ];

    // 插入列信息
    let allColumns = [];
    sections.forEach((section) => {
      allColumns = allColumns.concat(section.columns);
    });
    debugger;
    worksheet.columns = allColumns;

    // 插入标题行
    let headers = sections.map((section) => section.title);
    // 计算需要合并的列数
    let mergeLengths = sections.map((section) => section.columns.length);
    // 插入标题行
    worksheet.spliceRows(1, 0, headers);
    // 合并单元格并设置标题
    let startCol = 1;
    for (let i = 0; i < mergeLengths.length; i++) {
      let mergeLength = mergeLengths[i];
      if (mergeLength > 0) {
        let endCol = startCol + mergeLength - 1;
        worksheet.mergeCells(1, startCol, 1, endCol);
        worksheet.getCell(1, startCol).value = headers[i];
        startCol = endCol + 1;
      }
    }
  }
  // 添加内容
  // worksheet.addRow(dic);
  // worksheet.addRow(dic);
  // worksheet.addRow(dic);

  worksheet.eachRow(function (row, rowIndex) {
    row.eachCell({ includeEmpty: true }, function (cell, colNum) {
      cell.alignment = { vertical: "middle", horizontal: "center" };
      if (colNum >= 5 && rowIndex == 2) {
        cell.border = {
          bottom: { style: "thin", color: { argb: "eeeeee" } },
          top: { style: "thin", color: { argb: "000000" } },
          left: { style: "thin", color: { argb: "000000" } },
          right: { style: "thin", color: { argb: "000000" } },
        };
      } else {
        cell.border = {
          // top: { style: "thin", color: { argb: "000" } },
          left: { style: "thin", color: { argb: "000" } },
          bottom: { style: "thin", color: { argb: "000" } },
          right: { style: "thin", color: { argb: "000" } },
        };
      }

      cell.font = { bold: true };
    });
  });

  const fileName = "隐患排查数据.xlsx";
  const buffer = await workbook.xlsx.writeBuffer();
  const blob = new Blob([buffer], {
    type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
  });
  const link = document.createElement("a");
  link.href = URL.createObjectURL(blob);
  link.download = fileName;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}
</script>
<style>
</style>
