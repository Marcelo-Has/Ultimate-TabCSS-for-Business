<script>
	import Fa from 'svelte-fa';
	let color = '#850DF5';
	export let text = undefined;
	export let icon = undefined;
	export let classes = 'off';
	export let iSize = '20';

	if (classes == 'on' || classes == 'menu-on') {
		color = '#fff';
	}
</script>

<button class={classes}>
	{#if icon}
		<Fa {icon} size={iSize} {color} />
	{/if}
	{#if text}
		{text}
	{/if}
</button>

<style>
	.on {
		border: 1px solid var(--purple);
		background-color: var(--purple);
		color: var(--white);
		box-shadow: 0 4px var(--off-white2);
	}
	.off {
		border: 1px solid var(--white);
		background-color: var(--white);
		color: var(--purple);
		box-shadow: 0 4px var(--off-white2);
	}
	.off:hover,
	.menu-off:hover {
		background-color: var(--off-white);
		color: var(--purple);
		border: 1px solid var(--purple);
	}
	.on:hover,
	.menu-on:hover {
		background-color: var(--medium-purple);
		border: 1px solid var(--purple);
	}

	.menu-on {
		border: 1px solid var(--light-purple);
		background-color: var(--dark-purple);
		color: var(--white);
		font-weight: 500;
		height: 100%;
	}
	.menu-off {
		border: 1px solid var(--light-purple);
		background-color: var(--light-purple);
		color: var(--purple);
		font-weight: 500;
		height: 100%;
	}

	.on:active,
	.off:active {
		transform: translateY(4px);
		box-shadow: 0 2px #999;
	}
</style>
