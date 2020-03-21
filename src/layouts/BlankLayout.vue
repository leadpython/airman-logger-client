<template>
  <div class="blank-layout">
    <div style="position: absolute; left: 0; top: 0; width: 50vw; height: 100vh; display: flex; justify-content: center; align-items: center;">
      <img style="height: 500px; width: 500px; opacity: 0.25;" src="./../assets/background.png" />
    </div>
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
