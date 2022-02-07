<template>
	<div class="app-loader-wrapper">
		<div class="app-loader" />
	</div>
</template>

<script>
	export default {
		name: "AppLoader",
	};
</script>

<style scoped>
	.app-loader {
		font-size: 8px;
		border-radius: 50%;
		-webkit-animation: spin 2s linear infinite; /* Safari */
		animation: spin 2s linear infinite;

		border: .3em solid lightgreen;
		border-top: .3em solid gray;
		width: 4em;
		height: 4em;
	}

	.app-loader-wrapper {
		display: flex;
		align-items: center;
		justify-content: center;
	}

	@-webkit-keyframes spin {
		0% {
			-webkit-transform: rotate(0deg);
		}
		100% {
			-webkit-transform: rotate(360deg);
		}
	}

	@keyframes spin {
		0% {
			transform: rotate(0deg);
		}
		100% {
			transform: rotate(360deg);
		}
	}
</style>
