<template>
	<drag-drop-template title="LOGO'S" @action="evt => $emit('action', evt)">
		<div class="component__image">
			<media-upload 
				@changeImage="pushImage"
				@deleteImage="deleteImage"
				v-for="(image, index) in imageList"
				:key="image.id"
				class="component__image-logo"
				v-model="images[index]"
			/>
		</div>
	</drag-drop-template>
</template>

<script>
	import dragDropTemplate from '../../base';
	import mediaUpload from '@/components/mediaUpload';

	export default {
		components: {
			dragDropTemplate,
			mediaUpload,
		},
		props: {
			value: {
				required: false,
			},
		},
		data() {
			return {
				images: [],
				files: [],
				imageList: [0, ],
				counter: 0,
			};
		},
		methods: {
			pushImage( value ) {
				this.imageList.push( value );
				this.files = [];

				// For multiple images
				for( let file of this.imageList ) {
					if( file instanceof File ) {
						let formData = new FormData();
						formData.append( 'file', file );

						// Uploads the image
						this.$http.post( '/api/upload/image', formData ).then( response => {
							if( response.status === 200 ) {
								this.files.push( response.data );
							} else {
								console.error( response );
							}
						});
					}
				}

				this.$emit( 'input', this.files );
			},
			deleteImage() {
				for( let i = 0; i < this.images.length; i++ ) {
					if( this.images[i] === '' ) {
						this.images.splice( i, 1 );
						this.imageList.splice( i, 1 );
						this.files.splice( i, 1 );
						return;
					}
				}
			},
		},
		created() {
			if( this.value !== '' ) {
				this.images = this.value;

				for( let i = 0; i < this.images.length; i++ ) {
					this.imageList.push( 0 );
				}
			}
		},
	};
</script>

<style lang="scss" scoped>
	@import '../../../../../assets/sass/cms-main.scss';

	.component {
		&__image {
			display: flex;
			flex-flow: row wrap;

			&-logo {
				margin-right: 20px;
			}
		}
	}
</style>
