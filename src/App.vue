
<template>
  <main>
    <setDateRule class="p-all-5" />
  </main>
</template>
<script setup>
import setDateRule from "./components/setDateRule.vue";
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
@media (max-width: 500px) {
  .arco-picker-range-wrapper {
    flex-direction: column;
  }
}

/* 在屏幕宽度大于500px时应用以下样式 */
@media (min-width: 501px) {
  .arco-picker-range-wrapper {
    flex-direction: row;
  }
}
</style>
