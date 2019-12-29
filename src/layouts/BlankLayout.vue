<template>
  <div class="blank-layout">
    <router-view />
  </div>
</template>

<script>
export default {
  name: 'BlankLayout',

  data () {
    return {
    }
  }
}
</script>

<style lang="scss">
.blank-layout {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
