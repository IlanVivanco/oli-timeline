<script setup>
import { reactive, computed } from 'vue';

const LOCALE = 'de';
const DAY_IN_MILLI = 1000 * 60 * 60 * 24;
const HOUR_IN_MILLI = 1000 * 60 * 60;
const MINUTE_IN_MILLI = 1000 * 60;
const SECOND_IN_MILLI = 1000;
const DAYS_PER_WEEK = 7;

const props = defineProps({
	timelineData: {
		type: Array,
		required: true,
		default: () => [],
	},
	reversed: {
		type: Boolean,
		default: true,
	},
});

const peopleData = reactive(
	props.timelineData.map((person) => ({
		...person,
		datailsOpen: false,
	}))
);

const sortedData = computed(() => {
	const data = peopleData;

	data.sort((a, b) => {
		return new Date(a.date) - new Date(b.date);
	});

	return props.reversed ? data.reverse() : data;
});

function changeDetailStatus(person) {
	person.datailsOpen = !person.datailsOpen;
}

function UTCDate(date) {
	return new Date(date).toISOString();
}

function getDaysTilBirthday(date) {
	const birthday = new Date(date);
	const today = new Date();

	birthday.setHours(0, 0, 0, 0);
	birthday.setFullYear(today > birthday ? today.getFullYear() + 1 : today.getFullYear());

	return Math.floor((birthday - today) / this.DAY_IN_MILLI);
}

function getDateUnits(date) {
	const birthday = new Date(date);
	const today = new Date();
	const diffTime = Math.abs(today - birthday);
	const diffInDays = Math.ceil(diffTime / this.DAY_IN_MILLI);
	const diffInHours = Math.ceil(diffTime / this.HOUR_IN_MILLI);
	const diffInMinutes = Math.ceil(diffTime / this.MINUTE_IN_MILLI);
	const diffInSeconds = Math.ceil(diffTime / this.SECOND_IN_MILLI);
	const diffInWeeks = Math.floor(diffInDays / this.DAYS_PER_WEEK);
	const diffInWeeksReminder = diffInDays % this.DAYS_PER_WEEK;

	return [
		['Semanas', `${diffInWeeks.toLocaleString(this.LOCALE)} semanas y ${diffInWeeksReminder} días`],
		['Días', `${diffInDays.toLocaleString(this.LOCALE)} días`],
		['Horas', `${diffInHours.toLocaleString(this.LOCALE)} horas`],
		['Minutos', `${diffInMinutes.toLocaleString(this.LOCALE)} minutos`],
		['Segundos', `${diffInSeconds.toLocaleString(this.LOCALE)} segundos`],
	];
}

function localizedDate(date) {
	const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' };
	return new Date(date).toLocaleDateString('es-ES', options);
}
</script>

<template>
	<div v-if="sortedData.length" class="timeline">
		<div v-for="person in sortedData" :key="person.name" class="timeline__item">
			<div
				class="content flex flex-col items-center px-4 py-10 max-w-sm bg-white rounded-lg border border-slate-200 shadow-md"
			>
				<img
					class="mb-3 w-24 h-24 rounded-full shadow-lg"
					:src="person.photo ? `/images/people/${person.photo}` : '/images/person.jpg'"
					:alt="person.name"
				/>
				<h2 v-if="person.name?.length" class="mb-1 text-xl font-semibold text-slate-700">
					{{ person.name }}
				</h2>
				<time
					v-if="person.date?.length"
					class="text-sm font-medium mb-4 text-slate-500"
					:datetime="UTCDate(person.date)"
				>
					{{ localizedDate(person.date) }}
				</time>
				<div class="flex mt-4 space-x-3 md:mt-6">
					<a
						v-if="!person.datailsOpen"
						href="#"
						class="inline-flex items-center text-sm font-medium text-center text-emerald-600"
						@click.prevent="changeDetailStatus(person)"
						>Más info</a
					>
					<a
						v-else
						href="#"
						class="inline-flex items-center text-sm font-medium text-center text-slate-400"
						@click.prevent="person.datailsOpen = !person.datailsOpen"
						>Cerrar</a
					>
				</div>

				<Transition name="bounce">
					<div v-if="person.datailsOpen" class="px-4 w-full mt-4 relative">
						<table class="shadow-md w-full text-sm text-left text-slate-500 sm:rounded-sm">
							<caption class="pb-6 text-lg font-semibold text-left text-slate-900 bg-white">
								<p class="mt-1 text-sm font-normal text-center text-slate-500">
									¡Faltan
									<strong>{{ getDaysTilBirthday(person.date) }}</strong> días para su cumpleaños!
								</p>
							</caption>
							<tbody>
								<tr
									v-for="[label, content] in getDateUnits(person.date)"
									:key="label"
									class="border-b border-slate-100"
								>
									<th scope="row" class="py-4 px-6 font-medium text-slate-900 whitespace-nowrap bg-slate-50">
										{{ label }}
									</th>
									<td class="py-4 px-6 text-right">{{ content }}</td>
								</tr>
							</tbody>
						</table>
					</div>
				</Transition>
			</div>
		</div>
	</div>
</template>

<style scoped lang="scss">
.bounce-enter-active {
	animation: bounce-in 0.6s;
}

.bounce-leave-active {
	animation: bounce-in 0.6s reverse;
}

@keyframes bounce-in {
	0% {
		transform: scale(0);
		opacity: 0;
	}
	75% {
		transform: scale(1.1);
	}
	100% {
		transform: scale(1);
		opacity: 1;
	}
}

.timeline {
	position: relative;
	max-width: 1200px;
	width: 100%;
	margin: 0 auto;

	&::after {
		content: '';
		position: absolute;
		width: 6px;
		background-color: var(--color-emerald-600);
		top: 0;
		bottom: 0;
		left: 90%;
		margin-left: -3px;

		@media screen and (min-width: 1024px) {
			left: 50%;
		}
	}

	&__item {
		padding: 10px 40px;
		position: relative;
		background-color: inherit;
		width: 90%;
		min-width: 500px;

		@media screen and (min-width: 1024px) {
			width: 50%;
		}

		.content {
			position: relative;
		}

		&::after {
			content: '';
			position: absolute;
			width: 25px;
			height: 25px;
			right: 0;
			background-color: var(--color-white);
			border: 6px solid var(--color-emerald-600);
			top: 50%;
			transform: translate(50%, -50%);
			border-radius: 50%;
			z-index: 1;
		}

		&:nth-child(odd) {
			left: 0;
			@media screen and (min-width: 520px) {
				&::before {
					content: '';
					position: absolute;
					top: 50%;
					right: 0;
					width: 50%;
					transform: translateY(-50%);
					z-index: 0;
					height: 6px;
					background-color: var(--color-emerald-600);
				}
			}
			.content {
				@media screen and (min-width: 520px) {
					margin: 0 0 0 60px;

					&::before {
						content: '';
						position: absolute;
						height: 15px;
						width: 15px;
						top: 50%;
						transform: translateY(-50%);
						right: -5px;
						background-color: var(--color-emerald-600);
						border: 6px solid var(--color-emerald-600);
						border-radius: 50%;
						z-index: 1;
					}
				}
			}
		}

		&:nth-child(even) {
			@media screen and (min-width: 520px) {
				&::before {
					content: '';
					position: absolute;
					top: 50%;
					transform: translateY(-50%);
					right: 0;
					width: 50%;
					z-index: 0;
					height: 6px;
					background-color: var(--color-emerald-600);
					@media screen and (min-width: 1024px) {
						left: 0;
					}
				}

				&::after {
					left: auto;
					right: 0;
					transform: translate(50%, -50%);
				}
			}

			@media screen and (min-width: 1024px) {
				left: 50%;

				&::after {
					left: 0;
					right: auto;
					transform: translate(-50%, -50%);
				}
			}

			.content {
				@media screen and (min-width: 520px) {
					margin: 0 60px 0 0;

					&::before {
						content: '';
						position: absolute;
						height: 15px;
						width: 15px;
						top: 50%;
						transform: translateY(-50%);
						right: -5px;
						background-color: var(--color-emerald-600);
						border: 6px solid var(--color-emerald-600);
						border-radius: 50%;
						z-index: 1;
						@media screen and (min-width: 1024px) {
							left: -5px;
						}
					}
				}

				@media screen and (min-width: 1024px) {
					margin: 0 0 0 60px;
				}
			}

			&::after {
				&::after {
					left: 0;
					transform: translate(-50%, -50%);
				}
			}
		}
	}
}
</style>
