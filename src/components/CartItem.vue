<template>
	<article class="cart-item">
		<header class="cart-item__header">
			<h2 class="cart-item__title">{{ title }}</h2>
			<span
					v-if="discount"
					class="cart-item__discount-label"
			>{{ discount.labelName }}: {{ discount.value }}{{ discount.type }}</span>
		</header>
		<label for="quantity" class="cart-item__price">
			<input
					id="quantity"
					class="cart-item__quantity"
					type="number"
					:value="quantity"
					min="1"
					@input="onUpdateHandler"
			/> x {{ CURRENCY_DEFAULT }}{{ price }}
			<span class="cart-item__prices">
				<span class="cart-item__subtotal" :class="discount ? 'cart-item__subtotal--with-discount' : ''">
					{{ CURRENCY_DEFAULT }}{{ (quantity * price).toFixed(2) }}
				</span>
				<span v-if="discount" class="cart-item__subtotal">
					{{ CURRENCY_DEFAULT }}{{ ((quantity * price) - discount.amount).toFixed(2) }}
				</span>
			</span>
		</label>
		<button
				class="cart-item__button"
				@click="$emit('onRemove', id)"
		>Remove from cart</button>
	</article>
</template>

<script>
	import { mixinCurrency } from "../mixins/currency";


	export default {
		name: "CartItem",
		mixins: [mixinCurrency],
		props: {
			id: {
				type: String,
				required: true,
			},
			title: {
				type: String,
				required: true,
			},
			price: {
				type: Number,
				required: true,
			},
			quantity: {
				type: Number,
				required: true,
			},
			discount: {
				type: Object,
			},
		},
		methods: {
			onUpdateHandler( event ) {
				let value = event.target.value;

				value = Math.abs( value );
				this.$emit( 'onUpdate', this.id, value );
			},
		},
		onUpdate: {
			event: "onUpdate"
		},
		onRemove: {
			event: "onRemove"
		},
	};
</script>

<style scoped>
	h2 {
		font-size: 16px;
		text-align: left;
	}

	.cart-item {
		padding: 20px;
		border-bottom: 1px solid #ccc;
	}

	.cart-item__header {
		display: flex;
		margin-bottom: 20px;
	}

	.cart-item__header h2 {
		margin: 0;
	}

	.cart-item__discount-label {
		margin-left: auto;
		padding: 2px 20px;
		border-radius: 15px;
		background-color: lightgreen;
		font-weight: bold;
	}

	.cart-item:last-child {
		border-bottom: none;
	}

	.cart-item__prices {
		margin-left: auto;
	}

	.cart-item__price {
		display: flex;
		align-items: flex-start;
		justify-content: flex-start;
	}

	.cart-item__subtotal--with-discount {
		text-decoration: line-through;
		margin-right: 10px;
		opacity: .7;
	}

	.cart-item__quantity {
		margin-right: 10px;
		width: 30px;
	}

	.cart-item__subtotal {
		font-weight: bold;
		text-align: right;
	}
</style>
