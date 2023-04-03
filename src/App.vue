<template>


  <my-excel :rows="rows" :cols="cols" />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import MyExcel from "./components/MyExcel.vue";
export default defineComponent({
  components: { MyExcel },
  data() {
    return {
      cols: 20,
      rows: 25,
    };
  },
});
</script>

<style lang="scss" scoped>

// .wrapper {
//   width: max-content;
// }
</style>
