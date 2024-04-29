<script lang="ts">
	import '../app.postcss';
	import { AppShell, AppBar } from '@skeletonlabs/skeleton';
</script>

<!-- App Shell -->
<AppShell>
	<svelte:fragment slot="header">
		<!-- App Bar -->
		<AppBar>
			<svelte:fragment slot="lead">
				<strong class="md:text-3xl">Cache Simulator</strong>
			</svelte:fragment>
			<svelte:fragment slot="trail">
				<a
					class="btn btn-sm variant-ghost-surface"
					href="https://frenzyvjn.tech"
					target="_blank"
					rel="noreferrer"
				>
					Vijayan
				</a>
				<a
					class="btn btn-sm variant-ghost-surface"
					href="https://frenzyvjn.tech"
					target="_blank"
					rel="noreferrer"
				>
					
				</a>
				<a
					class="btn btn-sm variant-ghost-surface"
					href=""
					target="_blank"
					rel="noreferrer"
				>
					
				</a>
			</svelte:fragment>
		</AppBar>
	</svelte:fragment>
	<!-- Page Route Content -->
	<slot />
</AppShell>
