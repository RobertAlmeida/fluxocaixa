<template>
  <div class="home">
    <Top msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
// @ is an alias to /src
import Top from '@/components/Top.vue'

export default {
  name: 'Home',
  components: {
    Top
  }
}
</script>
