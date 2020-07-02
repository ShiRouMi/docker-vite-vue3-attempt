<template>
  <div>
    {{count}}
  </div>
</template>

<script>
import { reactive, ref, computed } from 'vue'
export default {
  setup() {
    const count = ref(1)
    const plusOne = computed(() => count.value + 1)
    // const plusOne = computed({
    //   get: () => count.value + 1,
    //   set: val => count.value = val -1
    // })
    // plusOne.value = 1
    console.log('count.value = ' + count.value)
    console.log('plusOne.value = ' + plusOne.value)
  }
}
</script>

<style>

</style>