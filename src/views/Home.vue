<template>
  <div class="home container">
    <img style="width:150px" src="../assets/logo.jpg">
    <Mint />
  </div>
</template>

<script>
// @ is an alias to /src
import Mint from '@/components/Mint.vue'

export default {
  name: 'Home',
  components: {
    Mint
  }
}
</script>
