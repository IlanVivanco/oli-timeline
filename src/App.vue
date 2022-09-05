<template>
	<Timeline :timelineData="people" />
</template>

<script>
import Timeline from './components/Timeline.vue'
import people from './data/people'

export default {
  name: 'App',
  data:()=>({
    people
  }),
  components: {
    Timeline
  },
}
</script>

<style>
#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	text-align: center;
	color: #2c3e50;
	margin-top: 60px;
}
</style>
