<template>
	<cms-page>
		<template slot="title">
			<div class="order-title">
				Orders
			</div>
		</template>
		<template slot="actions"></template>
		<template>
			<pop-up v-model="dialog" color="red" button-color="red" @agree="agreeOnError()">
				<template slot="title">{{ errorTitle }}</template>
				<template slot="text">{{ errorMessage }}</template>
				<template slot="button">Reload Page</template>
			</pop-up>
			<div class="container-vuetable">
				<v-radio-group v-model="radiowrapper" row class="radio-wrapper">
					<v-radio label="All" value="all" class="radio-wrapper--all" color="$darkgray--light" @change="filterData('all')"/>
					<v-radio label="Business" value="business" color="$darkgray--light" @change="filterData('business')"/>
					<v-radio label="Customer" value="customer" color="$darkgray--light" @change="filterData('customer')"/>
				</v-radio-group>
				<v-data-table
					v-bind:headers="headers"
					v-bind:items="orderItems"
					three-line="true"
					v-bind:pagination.sync="pagination"
					class="elevation-1"
					rows-per-page-text="Results per page"
				>
					<template slot="headerCell" slot-scope="orderProps">
						<span>{{ orderProps.header.text }}</span>
					</template>
					<template slot="items" slot-scope="orderProps">
					<router-link :to="{name: 'orderdetail.index', params : { id: orderProps.item.id }}" tag="tr" >
						<td class="text-xs-left number-field">{{ orderProps.item.id }}</td>
						<td class="text-xs-left client-field">{{ orderProps.item.customer.firstName + ' ' + orderProps.item.customer.lastName}}</td>
						<td class="text-xs-left">{{ formatDate(orderProps.item.createdAt ) }}</td>
						<td class="text-xs-left">&euro; {{ formatPrice(orderProps.item.total) }}</td>
						<td v-if="orderProps.item.status === 'completed'"
							class="text-xs-left status--completed">{{ orderProps.item.status }}<cms-icon icon="check" color="green" class="status--icon"/></td>
						<td v-else-if="orderProps.item.status === 'cancelled'"
							class="text-xs-left status--canceled">{{ orderProps.item.status }}<cms-icon icon="close" color="red" class="status--icon"/></td>
						<td v-else class="text-xs-left">{{ orderProps.item.status }}</td>
						<td class="text-xs-left">
							<div class="table__action-buttons">
								<ftsf-button icon="edit" color="green" :to="{name: 'orderdetail.index', params : { id: orderProps.item.id }}"/>
								<ftsf-button @click.native="openPopUp(orderProps.item.id)" v-if="orderProps.item.status === 'completed' || orderProps.item.status === 'cancelled'" icon="trash" color="red"/>
								<ftsf-button @click.native="openCancelPopUp(orderProps.item.id)" v-else icon="close" color="red"/>
								<pop-up v-model="open" color="red" button-color="red" @agree="deleteOrder()">
									<template slot="title">DELETE ORDER</template>
									<template slot="text">Are you sure you want to delete this order?
									</template>
									<template slot="button">Yes</template>
								</pop-up>
								<pop-up v-model="openCancel" color="red" button-color="red" @agree="cancelOrder()">
									<template slot="title">CANCEL ORDER</template>
									<template slot="text">Are you sure you want to cancel this order?
									</template>
									<template slot="button">Yes</template>
								</pop-up>
							</div>
						</td>
						</router-link>
					</template>
				</v-data-table>
				<div class="orders-pagination">
					<v-pagination v-if="setPagination > 1" v-model="pagination.page" :length="setPagination" :total-visible="7" :color="null" />
				</div>
			</div>
		</template>
	</cms-page>
</template>

<script>
	import CmsPage from '@/components/page';
	import cmsIcon from '@/components/icon';
	import ftsfButton from '@/components/button';
	import popUp from '@/components/pop-up';

	export default {
		created() {
			this.getOrders();
		},
		data: function() {
			return {
				targetId: '',
				dialog: false,
				radiowrapper: 'all',
				errorTitle: '',
				errorMessage: '',
				pagination: {},
				selected: [],
				orderItems: [],
				open: false,
				openCancel: false,
				filter: [],
				headers: [
					{text: 'Number', value: 'id', class: 'table-number table-head', icon: 'expand_more'},
					{text: 'Client', value: 'customer.firstName', class: 'table-client table-head'},
					{text: 'Date', value: 'createdAt', class: 'table-head date-field'},
					{text: 'Total', value: 'total', class: 'table-head total-field'},
					{text: 'Status', value: 'status', class: 'table-head status-field'},
				],
			};
		},
		components: {
			CmsPage,
			cmsIcon,
			ftsfButton,
			popUp,
		},
		computed: {
			setPagination() {
				return this.pagination.rowsPerPage ? Math.ceil( this.pagination.totalItems / this.pagination.rowsPerPage ) : 0;
			},
		},
		methods: {
			agreeOnError() {
				window.location.reload();
			},
			formatPrice( value ) {
				let locale = 'nl';
				let options = {
					minimumFractionDigits: 2,
					maximumFractionDigits: 2,
				};
				let formatter = new Intl.NumberFormat( locale, options );
				return formatter.format( value );
			},
			formatDate( date ) {
				let month = '' + ( date.getMonth() + 1 );
				let day = '' + date.getDate();
				let year = date.getFullYear();

				if( month.length < 2 ) {
					month = '0' + month;
				}

				if( day.length < 2 ) {
					day = '0' + day;
				}
				return [ day, month, year, ].join( '-' );
			},
			filterData( x ) {
				this.orderItems = [];

				if( x === 'all' ) {
					this.orderItems = this.filter;
				} else if( x === 'business' ) {
					for( let i = 0; i < this.filter.length; i++ ) {
						if( this.filter[i].customer.isCorporate === true ) {
							this.orderItems.push( this.filter[i]);
						}
					}
				} else if( x === 'customer' ) {
					for( let i = 0; i < this.filter.length; i++ ) {
						if( this.filter[i].customer.isCorporate === false ) {
							this.orderItems.push( this.filter[i]);
						}
					}
				}
				this.pagination.totalItems = this.orderItems.length;
			},
			getOrders() {
				this.$http.get( '/api/order/' ).then( response => {
					this.filter = response.data;
					this.filter.forEach( ( element ) => {
						let formattedDate = new Date( element.createdAt.replace( /(\d{2})-(\d{2})-(\d{4})/, '$2/$1/$3' ) );
						element.createdAt = formattedDate;
					});

					this.filterData( 'all' );

					if( this.filter.length === 0 ) {
						this.dialog = true;
						this.errorTitle = response.status + ' ' + response.statusText;
						this.errorMessage = 'There are no orders available.';
					}
				}).catch( ( error ) => {
					if( error.response.status === 404 ) {
						this.dialog = true;
						this.errorTitle = error.response.status + ' ' + error.response.statusText;
						this.errorMessage = 'No orders were found.';
					} else {
						this.dialog = true;
						this.errorTitle = error.response.status + ' ' + error.response.statusText;
						this.errorMessage = 'Something went wrong!';
					}
				});
			},
			deleteOrder() {
				this.$http.delete( '/api/order/' + this.targetId ).then( response => {
					if( response.status === 204 ) {
						this.getOrders();
						location.reload();
					}
				});
			},
			cancelOrder() {
				this.$http.patch( '/api/order/' + this.targetId, {'status': 'cancelled'}).then( () => {
					this.getOrders();
				});
			},
			openPopUp( id ) {
				this.open = true;
				this.targetId = id;
			},
			openCancelPopUp ( id ) {
				this.openCancel = true;
				this.targetId = id;
			},

		},
		mounted() {
			let tableHead = document.querySelectorAll( '.table-head' );
			let dropdownIcon = document.querySelector( '.input-group__icon-cb' );

			for( let i = 0; i < tableHead.length; i++ ) {
				tableHead[i].innerHTML = tableHead[i].innerHTML.replace( 'arrow_upward', 'expand_more' );
			}

			dropdownIcon.innerHTML = dropdownIcon.innerHTML.replace( 'arrow_drop_down', 'expand_more' );
		},
	};
</script>

<style lang="scss" scoped>
	@import '../../../assets/sass/cms-main.scss';
	td.text-xs-left {
		text-transform: capitalize;
	}

	td.text-xs-left.client-field {
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		max-width: 5vw;
	}

	.order-title {
		padding-top: 18px;
		font-family: Roboto, sans-serif;
		font-size: 22px;
		text-transform: uppercase;
		color: $background-menu;
	}

	.container-vuetable {
		max-height: calc(103vh - 179px);
		padding: 23px 20px 20px;
	}
</style>
