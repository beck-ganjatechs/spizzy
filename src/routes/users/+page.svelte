<script lang="ts">
	import { getImageURL } from '$lib/utils';
	import { onMount, tick, onDestroy } from 'svelte';
	import { lazyLoad } from '$lib/lazyLoad'; // Import the lazy load function
	import { Input } from '$lib/components/ui/input/index.js'; // Import the Input component
	import { page } from '$app/stores';
	import ScrollToTopButton from '$lib/components/ui/ScrollToTopButton.svelte';
	import Icon from '@iconify/svelte';
	import { animateMainStagger } from '$lib/animations';

	import { gsap } from 'gsap';
	import ScrollIndicator from '$lib/components/ui/ScrollIndicator.svelte';

	export let data: any;

	let hidden = true;

	let filter = ''; // Add a filter variable
	let showScrollToTop = false;

	const handleScroll = () => {
		const shouldShow = window.scrollY > 100;
		if (shouldShow !== showScrollToTop) {
			showScrollToTop = shouldShow;
			if (showScrollToTop) {
				gsap.to('.scroll-to-top-btn', {
					opacity: 1,
					y: 0,
					duration: 1,
					ease: 'power4.out'
				});
			} else {
				gsap.to('.scroll-to-top-btn', {
					opacity: 0,
					y: 20,
					duration: 1,
					ease: 'power4.out'
				});
			}
		}
	};

	onMount(async () => {
		await tick();
		animateMainStagger();
		window.addEventListener('scroll', handleScroll);

		if (typeof window !== 'undefined') {
			const { gsap } = await import('gsap');
			const { ScrollTrigger } = await import('gsap/ScrollTrigger');
			gsap.registerPlugin(ScrollTrigger);
			hidden = false;

			// Animate "book" on mount
			gsap.fromTo(
				'.animate-user',
				{ opacity: 0, y: -10 },
				{
					opacity: 1,
					y: 0,
					duration: 1,
					ease: 'power4.out'
				}
			);

			// Animate "book" on mount
			gsap.fromTo(
				'.animate-db',
				{ opacity: 0, x: -50 },
				{
					opacity: 1,
					x: 0,
					duration: 1,
					ease: 'power4.out'
				}
			);

			// input on mount
			gsap.fromTo(
				'.animate-input',
				{ opacity: 0, scale: 0.95 },
				{
					opacity: 1,
					scale: 1,
					duration: 1,
					ease: 'power4.out'
				}
			);

			// input on mount
			gsap.fromTo(
				'.animate-grid',
				{ opacity: 0, y: 20 },
				{
					opacity: 1,
					delay: 0.25,
					y: 0,
					duration: 3,
					ease: 'power4.out'
				}
			);

			document.querySelectorAll('.user-wrapper').forEach((el) => {
				lazyLoad(el as HTMLElement);
			});
			ScrollTrigger.refresh();

			const unsubscribe = page.subscribe(() => {
				ScrollTrigger.getAll().forEach((trigger) => trigger.kill());
				ScrollTrigger.refresh();
			});

			onDestroy(() => {
				unsubscribe();
				ScrollTrigger.getAll().forEach((trigger) => trigger.kill());
				window.removeEventListener('scroll', handleScroll);
			});
		}
	});

	onDestroy(() => {
		if (typeof window !== 'undefined') {
			import('gsap/ScrollTrigger').then(({ ScrollTrigger }) => {
				ScrollTrigger.getAll().forEach((trigger) => trigger.kill());
				window.removeEventListener('scroll', handleScroll);
			});
		}
	});

	// Reactive filteredUsers array based on the filter input
	$: filteredUsers = data.users.filter((user: any) =>
		user.username.toLowerCase().includes(filter.toLowerCase())
	);
</script>

<ScrollIndicator />

<div class={`${hidden ? 'opacity-0' : ''} animate-item mx-auto max-w-2xl`}>
	<div class="bg-base-100 mx-auto h-full w-full">
		<h1 class="flex items-center gap-2 text-7xl font-bold text-primary">
			<span class="animate-user">user</span>
			<span class="animate-db font-thin text-primary/50">db</span>
		</h1>
	</div>

	<!-- Add an input field to filter by username -->

	<div class="sticky top-[57px] z-50 mb-5 border-b bg-background pt-5">
		<div
			class="animate-input mb-5 flex items-center rounded-lg border bg-background bg-background px-2 focus-within:ring-1 focus-within:ring-foreground focus-within:ring-offset-1"
		>
			<Icon icon={`material-symbols:search`} class="h-7 w-7" />
			<Input
				type="text"
				class="border-none p-2 outline-none focus-visible:shadow-none focus-visible:ring-0 focus-visible:ring-offset-0"
				placeholder="Search users by username..."
				bind:value={filter}
			/>
		</div>

		<div class="mb-2 text-xl font-thin">
			<!-- Display the total count of filtered users -->
			{filter ? 'query matches' : 'total users'}: {filteredUsers.length}
		</div>
	</div>

	<!-- Display filtered users -->
	<div class="animate-grid grid grid-cols-1 gap-2 px-1 pb-40 sm:grid-cols-2 md:grid-cols-3">
		{#each filteredUsers as user}
			<a href={`/users/${user.id}`} class="user-wrapper group/userCard">
				<div
					class={`rounded-lg border p-3 shadow transition-all duration-300 md:hover:border-foreground`}
				>
					<div class="flex flex-row items-center gap-2">
						<div>
							<div class="h-10 w-10">
								<img
									class="transition-translate h-full w-full rounded-full border object-cover shadow duration-200 md:group-hover/userCard:scale-[102%]"
									alt={user.username}
									src={user?.avatar
										? getImageURL(user?.collectionId, user?.id, user?.avatar)
										: `https://ui-avatars.com/api/?name=${user?.username}&background=random`}
								/>
							</div>
						</div>

						<h1 class="truncate whitespace-nowrap text-sm">{user.username}</h1>
					</div>
				</div>
			</a>
		{/each}
	</div>
</div>

{#if showScrollToTop === true}
	<div class="flex justify-center">
		<ScrollToTopButton />
	</div>
{/if}
