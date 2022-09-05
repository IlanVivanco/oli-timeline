<template>
	<Timeline :timelineData="people" />
</template>

<script>
import Timeline from './components/Timeline.vue';
import people from './data/people';

export default {
	name: 'App',
	data: () => ({
		people,
	}),
	components: {
		Timeline,
	},
};
</script>