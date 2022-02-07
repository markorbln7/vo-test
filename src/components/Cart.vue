<template>
	<div class="cart">
		<div class="cart__response" v-if="isLoading">
			<AppLoader />
		</div>
		<div class="cart__response cart__response--error" v-else-if="isError">
			<h2>Something went wrong, please try again</h2>
			<button @click="fetchCartItems">Fetch Data</button>
		</div>
		<div v-else-if="dataFiltered" class="cart__body">
			<div class="cart__items">
				<div v-if="!!dataFiltered.length">
					<CartItem
							v-for="({
								id,
								title,
								price,
								quantity,
								discount,
							}) in dataFiltered"
							:key="id"
							:id="id"
							:title="title"
							:price="price"
							:quantity="quantity"
							:discount="discount"
							@onUpdate="onQuantityChange"
							@onRemove="onRemove"
					/>
				</div>
				<p
						v-else
						class="cart__no-items"
				>Your cart is empty</p>
			</div>
			<div class="cart__total">
				<h3 v-if="calculateTotalDiscount">
					Discount: {{ CURRENCY_DEFAULT }}{{ calculateTotalDiscount.toFixed(2)}}
				</h3>
				<h2>Total: {{ CURRENCY_DEFAULT }}{{ calculateTotal.toFixed(2) }}</h2>
			</div>
		</div>
	</div>
</template>

<script>
	import CartItem from "./CartItem";
	import { mixinCurrency } from "../mixins/currency";
	import { CART_ITEM_CATEGORIES } from "../constants/cartItemCategories";
	import {
		BREAKFAST_AND_DRINK,
		BREAKFAST_DOUBLE,
		DISCOUNTS,
	} from "../constants/discounts";
	import {
		calcPercentagesGetAmount,
		isEven,
	} from "../utils/number";
	import { getArrayItemIndexById } from "../utils/array";
	import AppLoader from "./common/AppLoader";


	export default {
		components: { AppLoader, CartItem },
		mixins: [mixinCurrency],
		data: function () {
			return {
				CART_ITEM_CATEGORIES,
				data: null,
				isError: false,
				isLoading: false,
				dataFiltered: null,
			};
		},
		methods: {
			onQuantityChange( id, quantity ) {
				const cartItemIndex = getArrayItemIndexById( id, this.dataFiltered );

				this.dataFiltered[ cartItemIndex ].quantity = +quantity;
				this.calculateDiscounts();
			},
			onRemove( id ) {
				this.dataFiltered = this.dataFiltered.filter( item => item.id !== id );
				this.calculateDiscounts();
			},
			calculateDiscounts() {
				const getBreakfastItems = this.dataFiltered.filter( item => item.category === CART_ITEM_CATEGORIES.BREAKFAST );
				const getDrinkItems = this.dataFiltered.filter( item => item.category === CART_ITEM_CATEGORIES.DRINK );

				if (
					!getBreakfastItems.length
					|| (
						getBreakfastItems.length === 1
						&& !getDrinkItems.length
					)
				) {
					if ( this.dataFiltered.some( item => item.discount ) ) {
						const allItemsWithDiscount = this.dataFiltered.filter( item => item.discount );

						this.removeDiscounts( allItemsWithDiscount );
					}
				}

				if ( isEven( getBreakfastItems.length ) ) {
					if ( getDrinkItems.some( item => item.discount ) ) {
						const allDrinksWithDiscount = getDrinkItems.filter( item => item.discount );
						this.removeDiscounts( allDrinksWithDiscount );
					}

					return this.setDiscounts(
						getBreakfastItems,
						BREAKFAST_DOUBLE,
					);
				} else {
					const evenBreakfastItems = getBreakfastItems.slice( 0, getBreakfastItems.length - 1 );
					const lastBreakfastItem = getBreakfastItems[ getBreakfastItems.length - 1 ];

					// Breakfast & Breakfast
					if ( evenBreakfastItems.length ) {
						this.setDiscounts(
							evenBreakfastItems,
							BREAKFAST_DOUBLE,
						);
					}

					// Breakfast & Drink
					if ( getDrinkItems.length ) {
						const itemsIdsForDiscount = [
							lastBreakfastItem,
							getDrinkItems[ 0 ],
						];

						this.setDiscounts(
							itemsIdsForDiscount,
							BREAKFAST_AND_DRINK,
						);
					}
				}
			},
			setDiscounts( data, type ) {
				return data.forEach( (
					{
						id,
						quantity,
						price,
					},
				) => {
					const cartItemIndex = getArrayItemIndexById( id, this.dataFiltered );

					this.dataFiltered[ cartItemIndex ].discount = {
						...DISCOUNTS[ type ],
						amount: calcPercentagesGetAmount( ( quantity * price ), DISCOUNTS[ type ].value ),
					};
				} );
			},
			removeDiscounts( data = this.dataFiltered ) {
				return data.forEach( ( { id } ) => {
					const cartItemIndex = getArrayItemIndexById( id, this.dataFiltered );

					this.dataFiltered[ cartItemIndex ].discount = null;
				} );
			},

			// For bigger apps this should be handled through vuex and modules, with axios
			async fetchCartItems() {
				this.isLoading = true;

				try {
					const { items } = await fetch( '/cart.json' ).then( response => response.json() );
					this.data = items;
					this.isError = false;
					this.dataFiltered = items.map( item => ( {
						...item,
						discount: null,
					} ) );

					this.calculateDiscounts();
				} catch ( error ) {
					this.data = null;
					this.isError = error;
				} finally {
					this.isLoading = false;
				}
			},
		},
		computed: {
			calculateTotal() {
				return this.dataFiltered.reduce( ( prev, next ) => {
					const discount = next.discount
						? next.discount.amount
						: 0
					;

					return +prev + ( ( +next.price * +next.quantity ) - discount );
				}, 0 );
			},
			calculateTotalDiscount() {
				return this.dataFiltered.reduce( ( prev, next ) => {
					const discount = next.discount
						? next.discount.amount
						: 0;

					return +prev + +discount;
				}, 0 );
			},
		},
		mounted() {
			this.fetchCartItems();
		}
	};
</script>

<style scoped>
	.cart__body {
		display: flex;
	}

	.cart__response {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 100%;
		margin: 50px 0;
	}

	.cart__response--error {
		flex-direction: column;
	}

	.cart__items {
		border-right: 1px solid #ccc;
		width: 400px;
	}

	.cart__no-items {
		text-align: center;
	}

	.cart__total {
		padding-left: 20px;
	}
</style>
