<template>
	<router-link :to="to">
		<button :class="this.classes">
			<cms-icon v-if="icon" class="button__svg" :icon="icon" color="white" />
			<span>{{ this.text }}</span>
		</button>
	</router-link>
</template>

<script>
	import cmsIcon from '@/components/icon';

	export default {
		components: {
			cmsIcon,
		},
		props: {
			icon: {
				type: String,
				default: '',
			},
			color: {
				type: String,
				default: 'green',
			},
			text: {
				type: String,
			},
			to: {
				default() {
					return {};
				},
				type: Object,
			},
		},
		computed: {
			small() {
				return !this.text;
			},
			classes() {
				const classes = {
					button: true,
					'button--small': this.small,
				};
				classes[`button--${this.color}`] = true;
				return classes;
			},
		},
	};
</script>

<style lang="scss">
	@import '../../sass/cms-main.scss';

	.button {
		min-width: 149px;
		height: 60px;
		padding: 17px 26px 19px 24px;
		margin-left: 10px;
		border-radius: 3px;
		text-transform: uppercase;
		color: $white;
		font-size: 14px;
		font-weight: bold;
		outline: 0;
		font-family: 'Roboto', sans-serif;

		&__svg {
			display: inline-block;
			vertical-align: middle;
			height: 43px;
			width: 43px;
			padding: 13px 20px 0 0;
			margin-top: -15px;
		}

		&--small {
			display: flex;
			align-content: center;
			justify-content: center;
			width: 35px;
			height: 35px;
			min-width: initial;
			padding: 0;
			opacity: .8;

			.button__svg {
				display: inline-block;
				vertical-align: middle;
				height: 17px;
				width: 19px;
				padding: 0;
				margin-top: 0;
			}
		}

		&--green {
			background-color: $green--light;

			&:hover {
				background-color: $green;
			}
			
			&:focus {
				background-color: $green--dark;
			}
		}

		&--red {
			background-color: $red--light;

			&:hover {
				background-color: $red;
			}
			
			&:focus {
				background-color: $red--dark;
			}
		}
		
		&--darkgray {
			background-color: $darkgray--light;

			&:hover {
				background-color: $darkgray;
			}
			
			&:focus {
				background-color: $darkgray--dark;
			}
		}

		&--purple {
			background-color: $purple;
			opacity: .5;
			width: 43px;
			height: 43px;

			&:hover {
				opacity: 1;
			}

			&:focus {
				background-color: $purple--dark;
				opacity: 1;
			}
		}

		&--regular {
			padding: 18px 24px 19px;
			background-color: $green--light;

			&:hover {
				background-color: $green;
			}
			
			&:focus {
				background-color: $green--dark;
			}
		}

		&--sidemenu {
			margin: 0;
		}

		&--white {
			padding-bottom: 18px;
			margin-right: 6px;
			border: 1px solid $green--light;
			color: $green--light;

			&:hover {
				border-color: $green;
				color: $green;
			}
			
			&:focus {
				border-color: $green--dark;
				color: $green--dark;
			}
		}

		&--disabled {
			background-color: $disabled;
		}
		
		&--login-green {
			padding: 20px 26px 19px 24px;
			background-color: $green--light;

			&:hover {
				background-color: $green;
			}
			
			&:focus {
				background-color: $green--dark;
			}
		}
	}
</style>
