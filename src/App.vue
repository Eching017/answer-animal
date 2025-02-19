<template>
  <div id="app">
    <Animal />
    <Hover />
  </div>
</template>

<script>
import Animal from './components/Animal.vue'
import Hover from './components/Hover.vue';

export default {
  name: 'App',
  components: {
    Animal,
    Hover
  }
}
</script>
