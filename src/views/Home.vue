<template>
  <div class="home">
    <img style="width:30%" src="../assets/logo.png">
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
