
<template>
  <a-config-provider :locale="locale">
    <main>
      <setDateRule class="p-all-5" />
    </main>
  </a-config-provider>
</template>
<script setup>
import setDateRule from "./components/setDateRule.vue";
// import testView from "./components/test.vue";
import zhCN from "@arco-design/web-vue/es/locale/lang/zh-cn";
import enUS from "@arco-design/web-vue/es/locale/lang/en-us";
import { ref } from "vue";
import { bitable } from "@lark-base-open/js-sdk";

const locale = ref(zhCN);
bitable.bridge.getLanguage().then((lang) => {
  if (lang == "zh") {
    locale.value = zhCN;
  } else {
    locale.value = enUS;
  }
});
</script>

<style>
@import "@/assets/superStyle/superCss.css";
@import "@/assets/superStyle/selfCss.css";

main {
  /* padding: 103px; */
}
.labelCss {
  width: 70px;
  text-align: center;
  flex-shrink: 0;
}
h4 {
  font-size: calc(1.275rem + 0.3vw);
  margin-bottom: 1rem;
}
code {
  font-size: 0.875em;
  color: #d63384;
  word-wrap: break-word;
}
/* 在屏幕宽度小于等于500px时应用以下样式 */
@media (max-width: 460px) {
  .arco-picker-range-wrapper {
    flex-direction: column;
  }
  .arco-table-th-title {
    font-size: 10px;
  }
  .arco-auto-tooltip-content {
    font-size: 10px;
  }
  .arco-table-td-content {
    font-size: 10px;
  }
  .arco-table-cell {
    padding: 9px 2px !important;
  }
}

/* 在屏幕宽度大于500px时应用以下样式 */
@media (min-width: 460px) {
  .arco-picker-range-wrapper {
    flex-direction: row;
  }
  .arco-table-th-title {
    font-size: 14px;
  }
  .arco-auto-tooltip-content {
    font-size: 14px;
  }
  .arco-table-td-content {
    font-size: 14px;
  }
  .arco-table-cell {
    padding: 9px 5px;
  }
}
body {
  background: #fff;
}

</style>
