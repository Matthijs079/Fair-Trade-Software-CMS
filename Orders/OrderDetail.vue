<template>
	<cms-page>
		<template slot="title">
			<div class="page__header">
				<div class="page__header--order">
					order #{{ this.$route.params.id }}
				</div>
				<div class="page__header--customer">
					customer
				</div>
			</div>
		</template>
		<template slot="actions">
			<pop-up v-model="dialog" color="red" button-color="red" @agree="agreeOnError()">
				<template slot="title">{{ errorTitle }}</template>
				<template slot="text">{{ errorMessage }}</template>
				<template slot="button">Ok</template>
			</pop-up>
			<ftsf-button icon="close" color="red" text="DISCARD" :to="{name: 'orders.index'}"/>
			<ftsf-button @click.native.stop="open = true" v-if="status === 'cancelled' || status === 'completed'" icon="trash" color="red" text="DELETE"/>
			<ftsf-button icon="save" color="green" text="SAVE" @click.native="openChangeStatusPopUp"/>
			<pop-up v-model="open" color="red" button-color="red" @agree="deleteOrder">
				<template slot="title">DELETE ORDER</template>
				<template slot="text">Are you sure you want to delete this order?</template>
				<template slot="button">Yes</template>
			</pop-up>
			<pop-up v-model="openChangeStatus" color="red" button-color="red" @agree="updateOrder">
				<template slot="title">CHANGE ORDER</template>
				<template slot="text">Are you sure you want to change the status of this order?</template>
				<template slot="button">Yes</template>
			</pop-up>
		</template>
		<template>
			<div class="container container-detailpage">
				<div class="container-table-small">
					<h2 class="title">order details</h2>
					<v-data-table
						v-bind:items="order"
						hide-actions
						hide-headers
						class="elevation-1"
						>
						<template slot="items" slot-scope="orderdetailProps">
							<tr>
								<td class="text-xs-left">Date of placement</td>
								<td class="text-xs-left">{{ orderdetailProps.item.createdAt }}</td>
							</tr>
							<tr>
								<td class="text-xs-left">Price</td>
								<td class="text-xs-left">&euro; {{ formatPrice(orderdetailProps.item.total) }}</td>
							</tr>
							<tr>
								<td class="text-xs-left">Bank name</td>
								<td class="text-xs-left">{{ 'Needs to be implemented' }}</td>
							</tr>
							<tr>
								<td class="text-xs-left">Bank address</td>
								<td class="text-xs-left">{{ 'Needs to be implemented' }}</td>
							</tr>
							<tr>
								<td class="text-xs-left">Shipping</td>
								<td class="text-xs-left">{{ 'Needs to be implemented' }}</td>
							</tr>
						</template>
					</v-data-table>
					<h2 class="title">client details</h2>
					<v-data-table
						v-bind:items="order"
						hide-actions
						hide-headers
						class="elevation-1"
						>
						<template slot="items" slot-scope="orderdetailProps">
							<tr>
								<td class="text-xs-left">Name</td>
								<td class="text-xs-left">{{ orderdetailProps.item.customer.firstName + ' ' + orderdetailProps.item.customer.lastName }}</td>
							</tr>
							<tr>
								<td class="text-xs-left">E-mail address</td>
								<td class="text-xs-left">{{ orderdetailProps.item.customer.email }}</td>
							</tr>
							<tr>
								<td class="text-xs-left">Phone number</td>
								<td class="text-xs-left">{{ orderdetailProps.item.customer.phoneNumber }}</td>
							</tr>
							<tr>
								<td class="text-xs-left">Address</td>
								<td class="text-xs-left">{{ orderdetailProps.item.billingAddress.street }}</td>
							</tr>
							<tr>
								<td class="text-xs-left">ZIP Code</td>
								<td class="text-xs-left">{{ orderdetailProps.item.billingAddress.zipcode }}</td>
							</tr>
							<tr>
								<td class="text-xs-left">City</td>
								<td class="text-xs-left">{{ orderdetailProps.item.billingAddress.city }}</td>
							</tr>
							<tr>
								<td class="text-xs-left">Country</td>
								<td class="text-xs-left">{{ orderdetailProps.item.billingAddress.country }}</td>
							</tr>
						</template>
					</v-data-table>
					<h2 class="title">delivery address</h2>
					<v-data-table
						v-bind:items="order"
						hide-actions
						hide-headers
						class="elevation-1"
						>
						<template slot="items" slot-scope="orderdetailProps">
							<tr>
								<td class="text-xs-left">Street address</td>
								<td class="text-xs-left">{{ orderdetailProps.item.deliveryAddress.street }}</td>
							</tr>
							<tr>
								<td class="text-xs-left">Zip code</td>
								<td class="text-xs-left">{{ orderdetailProps.item.deliveryAddress.zipcode }}</td>
							</tr>
							<tr>
								<td class="text-xs-left">City</td>
								<td class="text-xs-left">{{ orderdetailProps.item.deliveryAddress.city }}</td>
							</tr>
							<tr>
								<td class="text-xs-left">Country</td>
								<td class="text-xs-left">{{ orderdetailProps.item.deliveryAddress.country }}</td>
							</tr>
						</template>
					</v-data-table>
					<div class="container-productstable">
						<h2 class="title products-table-title">Products</h2>
						<v-data-table
							v-bind:items="title"
							hide-actions
							hide-headers
							class="product-title"
							>
							<template slot="items" slot-scope="orderdetailProps">
								<td class="product-title__name">{{ orderdetailProps.item.title }}</td>
								<td class="product-title__properties">{{ orderdetailProps.item.properties }}</td>
								<td class="product-title__amount">{{ orderdetailProps.item.amount }}</td>
								<td class="product-title__total">{{ orderdetailProps.item.total }}</td>
							</template>
						</v-data-table>
						<v-data-table
							v-bind:items="products"
							hide-actions
							hide-headers
							class="elevation-1"
							>
							<template slot="items" slot-scope="orderdetailProps">
								<td class="text-xs-left clamp"><v-avatar tile size="32px"><img class="image" :src="orderdetailProps.item.properties.image_path"></v-avatar>{{ orderdetailProps.item.properties.name }}</td>
								<td class="text-xs-left"><div class="table-item--color" v-bind:style="{ backgroundColor: orderdetailProps.item.properties.color}"></div></td>
								<td class="text-xs-left">{{ orderdetailProps.item.properties['clothing-size'] }}</td>
								<td class="text-xs-left">{{ orderdetailProps.item.pivot.quantity }}</td>
								<td class="text-xs-left text-xs-flex">&euro; {{  formatPrice(orderdetailProps.item.price * orderdetailProps.item.pivot.quantity) }}</td>
							</template>
						</v-data-table>
						<div class="table-total">
							<v-data-table
								v-bind:items="order"
								hide-actions
								hide-headers
								class="elevation-1"
								>
								<template slot="items" slot-scope="orderdetailProps">
									<td class="text-xs-left">Total (incl. VAT)</td>
									<td class="text-xs-left"></td>
									<td class="text-xs-left"></td>
									<td class="table-amount">{{ orderdetailProps.item.totalQuantity }}</td>
									<td class="text-xs-left total-amount">&euro; {{ formatPrice(orderdetailProps.item.total) }}</td>
								</template>
							</v-data-table>
						</div>
					</div>
				</div>
				<div class="sidebar">
					<h2 class="sidebar__title">STATUS</h2>
					<v-radio-group v-model="status" column>
						<v-radio
							label="In Progress"
							color="secondary"
							value="in progress">
						</v-radio>
						<v-radio
							label="Completed"
							color="secondary"
							value="completed">
						</v-radio>
						<v-radio
							label="Cancelled"
							color="secondary"
							value="cancelled">
						</v-radio>
					</v-radio-group>
				</div>
			</div>
		</template>
	</cms-page>
</template>

<script>
	import cmsPage from '@/components/page';
	import ftsfButton from '@/components/button';
	import popUp from '@/components/pop-up';

	export default {
		components: {
			cmsPage,
			ftsfButton,
			popUp,
		},
		created() {
			this.getOrder();
		},
		data() {
			return {
				title: [
					{title: 'name', properties: 'properties', amount: 'amount', total: 'total'},
				],
				order: [],
				products: [],
				status: '',
				openChangeStatus: false,
				open: false,
				openCancel: false,
				dialog: false,
				errorTitle: '',
				errorMessage: '',
			};
		},
		methods: {
			agreeOnError() {
				window.location.href = '../orders';
			},
			formatPrice( value ) {
				let locale = 'nl';
				let options = {
					minimumFractionDigits: 2, maximumFractionDigits: 2,
				};
				let formatter = new Intl.NumberFormat( locale, options );
				return formatter.format( value );
			},
			getOrder() {
				this.$http.get( '/api/order/' + this.$route.params.id ).then( response => {
					this.order.push( response.data );
					this.status = response.data.status;
					this.products = response.data.products;
				}).catch( ( error ) => {
					if( error.response.status === '404' ) {
						this.dialog = true;
						this.errorTitle = error.response.status + ' ' + error.response.statusText;
						this.errorMessage = 'No order found.';
					} else {
						this.dialog = true;
						this.errorTitle = error.response.status + ' ' + error.response.statusText;
						this.errorMessage = 'Somthing went wrong.';
					}
				});
			},
			deleteOrder() {
				this.$http.delete( '/api/order/' + this.$route.params.id ).then( () => {
					this.$router.push( '/orders' );
				}).catch( ( error ) => {
					if( error.response.status === '404' ) {
						this.dialog = true;
						this.errorTitle = error.response.status + ' ' + error.response.statusText;
						this.errorMessage = 'No order found.';
					} else {
						this.dialog = true;
						this.errorTitle = error.response.status + ' ' + error.response.statusText;
						this.errorMessage = 'Somthing went wrong.';
					}
				});
			},
			updateOrder() {
				this.$http.patch( '/api/order/' + this.$route.params.id, {'status': this.status}).then( () => {
					this.$router.push( '/orders' );
				}).catch( ( error ) => {
					if( error.response.status === '404' ) {
						this.dialog = true;
						this.errorTitle = error.response.status + ' ' + error.response.statusText;
						this.errorMessage = 'No order found.';
					} else {
						this.dialog = true;
						this.errorTitle = error.response.status + ' ' + error.response.statusText;
						this.errorMessage = 'Somthing went wrong.';
					}
				});
			},
			openChangeStatusPopUp() {
				this.openChangeStatus = true;
			},
		},
	};
</script>

<style lang='scss' scoped>

	@import '../../../assets/sass/cms-main.scss';

	.page__header {
		display: flex;
		flex-direction: row;
		font-family: Roboto, sans-serif;
		text-transform: uppercase;

		&--order {
			color: $background-menu;
			padding-top: 18px;
			font-size: 22px;
		}

		&--customer {
			color: $separator-overview;
			align-self: flex-end;
			font-weight: 900;
			font-size: 18px;
			padding: 0 0 2px 6%;
		}
	}

	page-content {
		width: 100% !important;
	}

	.container {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		padding: 0;
	}

	.container-detailpage {
		width: 100% !important; // vuetify overwrites the styling

		@media only screen and (min-width: 960px) {
			max-width: 100% !important; // vuetify overwrites the styling
		}
	}

	.sidebar {
		background-color: $white;
		border: 1px solid $darkgray--light;
		width: 32%;
		height: 190px;
		padding: 20px 30px;
		font-family: Roboto, sans-serif;
		font-weight: 500;
		overflow: hidden;
		text-overflow: ellipsis;

		&__title {
			border-bottom: 2px solid $darkgray--light;
			padding-bottom: 10px;
			color: $darkgray--dark;
			font-size: 1.4rem;
			font-weight: 700;
		}
	}

	.product-title {
		display: flex;
		flex-flow: row;
		list-style-type: none;

		&__name {
			width: 17vw !important; // vuetify overwrites the styling
			padding-left: 65px !important; // vuetify overwrites the styling
			font-weight: 700;
			font-size: 16px;
			text-transform: uppercase;

			@media screen and (min-width: 1903px) {
				width: 22vw !important; // vuetify overwrites the styling
			}
		}

		&__properties {
			font-weight: 700;
			font-size: 16px;
			text-transform: uppercase;
		}

		&__amount {
			padding-left: 4px !important; // vuetify overwrites the styling
			padding-right: 44px !important; // vuetify overwrites the styling
			font-weight: 700;
			font-size: 16px;
			text-transform: uppercase;

			@media screen and (min-width: 1903px) {
				padding-left: 2vw !important; // vuetify overwrites the styling
			}
		}

		&__total {
			display: flex;
			padding-right: 10px !important; // vuetify overwrites the styling
			align-items: center;
			font-weight: 700;
			font-size: 16px;
			text-transform: uppercase;

			@media screen and (min-width: 1903px) {
				justify-content: center;
			}
		}
	}

	td.text-xs-left.clamp {
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		max-width: 20%;
	}

	.products-table-title {
		margin-top: 50px !important; // vuetify overwrites the styling
	}

	.table-amount {
		padding-left: 49px !important; // vuetify overwrites the styling
	}

	.avatar--tile img {
		border-radius: 5px !important; // vuetify overwrites the styling
	}

	.total-amount {
		font-size: 16px !important; // vuetify overwrites the styling
	}
</style>
