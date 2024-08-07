<script setup>
import { reactive, computed, ref, onMounted, watchEffect } from 'vue';

const LOCALE = 'es-ES';

const DAY_IN_MILLI = 1000 * 60 * 60 * 24;
const HOUR_IN_MILLI = 1000 * 60 * 60;
const MINUTE_IN_MILLI = 1000 * 60;
const SECOND_IN_MILLI = 1000;
const DAYS_PER_WEEK = 7;
const DAYS_PER_YEAR = 365.25;  // Including leap years
const DAYS_PER_MONTH = DAYS_PER_YEAR / 12;

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

let peopleData = reactive(
	props.timelineData.map((person) => ({
		...person,
		datailsOpen: false,
		// formattedUnits: formatDateUnits(person.date),
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

	birthday.setFullYear(today.getFullYear());

	if (today >= birthday) {
		birthday.setFullYear(today.getFullYear() + 1);
	}

	return Math.floor((birthday - today) / DAY_IN_MILLI);
}

function getAge(date) {
	const givenDate = new Date(date);
	const today = new Date();

	let years = today.getFullYear() - givenDate.getFullYear();
	const monthDifference = today.getMonth() - givenDate.getMonth();
	const dayDifference = today.getDate() - givenDate.getDate();

	// Adjust the years if the current date is before the given date's anniversary this year
	if (monthDifference < 0 || (monthDifference === 0 && dayDifference < 0)) {
		years--;
	}

	return years;
}

function getRemainingMonths(date) {
	const givenDate = new Date(date);
	const today = new Date();

	let years = today.getFullYear() - givenDate.getFullYear();
	let months = today.getMonth() - givenDate.getMonth();
	const dayDifference = today.getDate() - givenDate.getDate();

	// Adjust the years if the current date is before the given date's anniversary this year
	if (months < 0 || (months === 0 && dayDifference < 0)) {
		years--;
	}

	// Calculate remaining months after adjusting years
	months = (today.getMonth() + 12 - givenDate.getMonth()) % 12;

	// Adjust if the day difference is negative
	if (dayDifference < 0) {
		months--;
	}

	return months;
}

function calculateDiffTime(date) {
	const birthday = new Date(date);
	const today = new Date();
	return Math.abs(today - birthday);
}

function getDiffInDays(diffTime) {
	return Math.ceil(diffTime / DAY_IN_MILLI);
}

function getDiffInHours(diffTime) {
	return Math.ceil(diffTime / HOUR_IN_MILLI);
}

function getDiffInMinutes(diffTime) {
	return Math.ceil(diffTime / MINUTE_IN_MILLI);
}

function getDiffInSeconds(diffTime) {
	return Math.ceil(diffTime / SECOND_IN_MILLI);
}

function getDiffInWeeks(diffInDays) {
	return Math.floor(diffInDays / DAYS_PER_WEEK);
}

function getDiffInWeeksReminder(diffInDays) {
	return diffInDays % DAYS_PER_WEEK;
}

function getDiffInYears(diffInDays) {
	return Math.floor(diffInDays / DAYS_PER_YEAR);
}

function getDiffInMonths(diffInDays) {
	return Math.floor(diffInDays / DAYS_PER_MONTH);
}

function localizedDate(date) {
	const options = { year: 'numeric', month: 'long', day: 'numeric' };
	return date.toLocaleString('es-ES', options);
}

function getDateUnits(date) {
	const diffTime = calculateDiffTime(date);
	const diffInDays = getDiffInDays(diffTime);
	const diffInHours = getDiffInHours(diffTime);
	const diffInMinutes = getDiffInMinutes(diffTime);
	const diffInSeconds = getDiffInSeconds(diffTime);
	const diffInWeeks = getDiffInWeeks(diffInDays);
	const diffInWeeksReminder = getDiffInWeeksReminder(diffInDays);
	const diffInYears = getDiffInYears(diffInDays);
	const diffInMonths = getDiffInMonths(diffInDays);

	return {
		diffInYears,
		diffInMonths,
		diffInWeeks,
		diffInWeeksReminder,
		diffInDays,
		diffInHours,
		diffInMinutes,
		diffInSeconds
	};
}

function formatDateUnits(people) {
	people.forEach(person => {
		const dateUnits = getDateUnits(person.date);
		const formattedDateUnits = [];

		for (let [key, value] of Object.entries(dateUnits)) {
			switch (key) {
				case 'diffInYears':
					formattedDateUnits.push(['Años', `${localizedDate(value)}`]);
					break;
				case 'diffInMonths':
					formattedDateUnits.push(['Meses', `${localizedDate(value)}`]);
					break;
				case 'diffInWeeks':
					formattedDateUnits.push(['Semanas', `${localizedDate(value)}`]);
					break;
				case 'diffInDays':
					formattedDateUnits.push(['Días', `${localizedDate(value)}`]);
					break;
				case 'diffInHours':
					formattedDateUnits.push(['Horas', `${localizedDate(value)}`]);
					break;
				case 'diffInMinutes':
					formattedDateUnits.push(['Minutos', `${localizedDate(value)}`]);
					break;
				case 'diffInSeconds':
					formattedDateUnits.push(['Segundos', `${localizedDate(value)}`]);
					break;
			}
		}

		person.formattedUnits = formattedDateUnits;
	});

	return people;
}

onMounted(() => {
	setInterval(() => {
		peopleData = formatDateUnits(peopleData);
	}, 1000);
});
</script>

<template>
	<div v-if="sortedData.length" class="timeline">
		<div v-for="person in sortedData" :key="person.name" class="timeline__item">
			<div class="content flex flex-col items-center px-4 py-10 max-w-sm bg-white rounded-lg border border-slate-200 shadow-md">
				<img class="mb-3 w-24 h-24 rounded-full shadow-lg" :src="person.photo ? `/images/people/${person.photo}` : '/images/person.jpg'" :alt="person.name" />
				<h2 v-if="person.name?.length" class="mb-1 text-xl font-semibold text-slate-700">
					{{ person.name }}
				</h2>
				<time v-if="person.date?.length" class="text-sm font-medium mb-4 text-slate-500" :datetime="UTCDate(person.date)">
					{{ localizedDate(person.date) }}
				</time>
				<div class="flex mt-4 space-x-3 md:mt-6">
					<a v-if="!person.datailsOpen" href="#" class="inline-flex items-center text-sm font-medium text-center text-emerald-600" @click.prevent="changeDetailStatus(person)">Más info</a>
					<a v-else href="#" class="inline-flex items-center text-sm font-medium text-center text-slate-400" @click.prevent="person.datailsOpen = !person.datailsOpen">Cerrar</a>
				</div>

				<Transition name="bounce">
					<div v-if="person.datailsOpen" class="px-4 w-full mt-4 relative">
						<table class="shadow-md w-full text-sm text-left text-slate-500 sm:rounded-sm">
							<caption class="pb-6 text-lg font-semibold text-left text-slate-900 bg-white">
								<p class="mt-1 text-sm font-normal text-center text-slate-500">
									Tiene <strong>{{ getAge(person.date) }}</strong> años
									<span v-if="getRemainingMonths(person.date) > 0">
										y <strong>{{ getRemainingMonths(person.date) }}</strong> meses
									</span>
								</p>
								<p class="mt-1 text-sm font-normal text-center text-slate-500">
									<span v-if="getDaysTilBirthday(person.date) > 0">
										¡Faltan
										<strong>{{ getDaysTilBirthday(person.date) }}</strong> días para
									</span>
									<span v-else>
										hoy es
									</span>
									su cumpleaños!
									<span v-if="getDaysTilBirthday(person.date) == 0">🥳</span>
								</p>
							</caption>
							<tbody>
								<tr v-for="[label, content] in person.formattedUnits" :key="label" class="border-b border-slate-100">
									<td class="py-4 px-6 text-right">{{ content }}</td>
									<th scope="row" class="py-4 px-6 w-6 font-medium text-slate-900 whitespace-nowrap bg-slate-50">
										{{ label }}
									</th>
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
		padding-top: 6rem;
		padding-bottom: 4rem;

		@media screen and (width < 600px) {
			padding-top: 2rem;
			width: fit-content;
		}

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

			@media screen and (width < 600px) {
				left: 50%;
				z-index: -1;
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

			@media screen and (width < 600px) {
				width: 100%;
			}

			.content {
				position: relative;

				@media screen and (width < 600px) {
					min-width: 100%;
					margin-bottom: 2rem;
				}
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

				@media screen and (width < 600px) {
					inset: auto;
					top: 0;
					transform: translate(-50%, 0);
				}
			}

			&:nth-child(odd) {
				left: 0;

				@media screen and (min-width: 600px) {
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
					@media screen and (min-width: 1024px) {
						margin: 0 0 0 60px;
					}

					@media screen and (min-width: 600px) {
						margin: 0 0 0 auto;
					}

					@media screen and (min-width: 600px) {
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
				@media screen and (min-width: 600px) {
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
					@media screen and (min-width: 1024px) {
						margin: 0 60px 0 0;
					}

					@media screen and (min-width: 600px) {
						margin: 0 0 0 auto;
					}

					@media screen and (min-width: 600px) {

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