<script lang="ts">
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
	import logo from '$lib/assets/logo.png';

	const REPO = 'VUXXE/Mesh';
	const APP_URL = import.meta.env.VITE_APP_URL || 'https://app-mesh.asy.web.id';

	let starCount = $state<number | null>(null);
	let isLight = $state(false);
	let activeSelfHostTab = $state<'docker' | 'bun' | 'deploy'>('docker');
	let activeEngineTab = $state<'vector' | 'storage' | 'presence' | 'security'>('vector');
	let copied = $state(false);
	let copyTimeout: ReturnType<typeof setTimeout> | null = null;

	let showLightbox = $state(false);
	let isScrolled = $state(false);
	let scrollContainer: HTMLElement | null = $state(null);

	function handleKeydown(e: KeyboardEvent) {
		if (e.key === 'Escape' && showLightbox) {
			showLightbox = false;
		}
	}

	function handleScroll() {
		const top = scrollContainer ? scrollContainer.scrollTop : window.scrollY;
		if (!isScrolled && top > 80) {
			isScrolled = true;
		} else if (isScrolled && top < 40) {
			isScrolled = false;
		}
	}

	function formatStars(n: number): string {
		if (n >= 1000) {
			const v = n / 1000;
			return `${v >= 100 ? Math.round(v) : v.toFixed(1).replace(/\.0$/, '')}k`;
		}
		return `${n}`;
	}

	function toggleTheme() {
		if (typeof document !== 'undefined') {
			isLight = document.documentElement.classList.toggle('light');
			try {
				localStorage.setItem('mesh_theme', isLight ? 'light' : 'dark');
			} catch {
				// Ignore storage errors
			}
		}
	}

	onMount(() => {
		if (typeof document !== 'undefined') {
			isLight = document.documentElement.classList.contains('light');
		}

		window.addEventListener('keydown', handleKeydown);
		window.addEventListener('scroll', handleScroll, { passive: true });

		try {
			const cached = localStorage.getItem('mesh_gh_stars');
			const cachedAt = Number(localStorage.getItem('mesh_gh_stars_at') ?? 0);
			if (cached !== null && Date.now() - cachedAt < 3600000) {
				starCount = Number(cached);
				return;
			}
		} catch {
			// Ignore storage errors
		}

		fetch(`https://api.github.com/repos/${REPO}`)
			.then((res) => (res.ok ? (res.json() as Promise<{ stargazers_count?: unknown }>) : null))
			.then((data) => {
				if (data && typeof data.stargazers_count === 'number') {
					starCount = data.stargazers_count;
					try {
						localStorage.setItem('mesh_gh_stars', String(starCount));
						localStorage.setItem('mesh_gh_stars_at', String(Date.now()));
					} catch {
						// Ignore storage errors
					}
				}
			})
			.catch(() => {
				// Offline or rate-limited
			});

		return () => {
			window.removeEventListener('keydown', handleKeydown);
			window.removeEventListener('scroll', handleScroll);
			if (copyTimeout) clearTimeout(copyTimeout);
		};
	});

	function generateRoomId(): string {
		const chars = 'abcdefghjkmnpqrstuvwxyz23456789';
		let result = '';
		for (let i = 0; i < 6; i++) {
			result += chars.charAt(Math.floor(Math.random() * chars.length));
		}
		return `room-${result}`;
	}

	function createRoom() {
		const newId = generateRoomId();
		if (typeof window !== 'undefined') {
			window.location.href = `${APP_URL}/room/${newId}`;
		}
	}

	const snippets = {
		docker: `git clone https://github.com/VUXXE/Mesh.git\ncd Mesh\ndocker compose up -d`,
		bun: `git clone https://github.com/VUXXE/Mesh.git\ncd Mesh\nbun install\nbun start`,
		deploy: `git clone https://github.com/VUXXE/Mesh.git\ncd Mesh\nbun install\nbun run deploy`
	};

	function copyCode(text: string) {
		if (navigator?.clipboard) {
			navigator.clipboard.writeText(text);
			copied = true;
			if (copyTimeout) clearTimeout(copyTimeout);
			copyTimeout = setTimeout(() => {
				copied = false;
			}, 2000);
		}
	}
</script>

<svelte:head>
	<title>Mesh: Open-Source Edge Vector Whiteboard</title>
	<meta
		name="description"
		content="Fast, distraction-free collaborative vector whiteboard on Cloudflare Workers and Svelte 5. Sub-16ms drawing feedback, embedded SQLite, and adaptive ephemeral presence."
	/>
</svelte:head>

<div
	bind:this={scrollContainer}
	onscroll={handleScroll}
	class="landing-dots relative h-screen w-screen overflow-x-hidden overflow-y-auto bg-(--surface-0) text-(--ink-1) selection:bg-(--accent-lime)/30"
>
	<!-- SIDE RAILS (OpenDesign signature technical rails) -->
	<aside
		class="side-rail left pointer-events-none fixed top-0 bottom-0 left-0 z-30 hidden w-10 items-center justify-center border-r border-(--surface-2)/60 xl:flex"
	>
		<span class="rail-text text-(--ink-3)">MESH // EDGE VECTOR ENGINE // ISOLATE SQLITE</span>
	</aside>
	<aside
		class="side-rail right pointer-events-none fixed top-0 right-0 bottom-0 z-30 hidden w-10 items-center justify-center border-l border-(--surface-2)/60 xl:flex"
	>
		<span class="rail-text text-(--ink-3)">SUB-16MS LATENCY // 15HZ ADAPTIVE // ZERO-IDLE</span>
	</aside>

	<!-- NAVIGATION BAR (Full to Pill on Scroll) -->
	<header
		class="sticky z-50 mx-auto w-full transition-all duration-500 ease-[cubic-bezier(0.16,1,0.3,1)] {isScrolled
			? 'top-4 max-w-5xl px-4 sm:px-6'
			: 'top-0 max-w-6xl px-4 sm:px-6 lg:px-8'}"
	>
		<nav
			class="flex items-center justify-between gap-4 transition-all duration-500 ease-[cubic-bezier(0.16,1,0.3,1)] {isScrolled
				? 'rounded-full border border-(--surface-2) bg-(--surface-1)/95 px-4 py-2.5 shadow-lg backdrop-blur-md sm:px-6'
				: 'rounded-none border-b border-(--surface-2) bg-(--surface-0)/90 px-2 py-3.5 backdrop-blur-sm sm:px-4'}"
		>
			<!-- Brand Mark & Identity -->
			<a href="/" class="text-decoration-none flex items-center gap-2.5">
				<img src={logo} alt="Mesh Logo" class="h-7 w-7 rounded-lg object-contain shadow-xs" />
				<span class="font-extrabold tracking-tight text-(--ink-1) sm:text-base">Mesh</span>
			</a>

			<!-- Nav Links -->
			<div class="hidden items-center gap-6 text-xs font-semibold tracking-wide md:flex">
				<a href="#workbench" class="text-(--ink-2) transition-colors hover:text-(--ink-1)">
					Workbench
				</a>
				<a href="#engine" class="text-(--ink-2) transition-colors hover:text-(--ink-1)"> Engine </a>
				<a href="#architecture" class="text-(--ink-2) transition-colors hover:text-(--ink-1)">
					Architecture
				</a>
				<a href="#self-host" class="text-(--ink-2) transition-colors hover:text-(--ink-1)">
					Self-Host
				</a>
				<a href="#tech-specs" class="text-(--ink-2) transition-colors hover:text-(--ink-1)">
					Specs
				</a>
			</div>

			<!-- Right Actions: Theme Toggle, GitHub Stars, Action Pill -->
			<div class="flex items-center gap-2 sm:gap-2.5">
				<!-- Theme Toggle -->
				<button
					type="button"
					onclick={toggleTheme}
					aria-label="Toggle visual theme"
					class="group flex h-8 w-8 items-center justify-center rounded-lg border border-(--surface-2) bg-(--surface-0)/80 text-(--ink-2) shadow-xs transition-all hover:border-(--surface-3) hover:bg-(--surface-1) hover:text-(--ink-1)"
				>
					{#if isLight}
						<!-- Moon Icon -->
						<svg
							class="h-4 w-4 transition-transform duration-300 group-hover:-rotate-12 group-hover:scale-110"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
							stroke-linecap="round"
							stroke-linejoin="round"
						>
							<path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path>
						</svg>
					{:else}
						<!-- Sun Icon -->
						<svg
							class="h-4 w-4 transition-transform duration-300 group-hover:rotate-45 group-hover:scale-110"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
							stroke-linecap="round"
							stroke-linejoin="round"
						>
							<circle cx="12" cy="12" r="5"></circle>
							<line x1="12" y1="1" x2="12" y2="3"></line>
							<line x1="12" y1="21" x2="12" y2="23"></line>
							<line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line>
							<line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line>
							<line x1="1" y1="12" x2="3" y2="12"></line>
							<line x1="21" y1="12" x2="23" y2="12"></line>
							<line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line>
							<line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line>
						</svg>
					{/if}
				</button>

				<!-- GitHub Pill (Divided Badge) -->
				<a
					href="https://github.com/{REPO}"
					target="_blank"
					rel="noreferrer"
					class="group hidden h-8 items-center gap-2 rounded-lg border border-(--surface-2) bg-(--surface-0)/80 px-2.5 text-xs font-semibold text-(--ink-2) shadow-xs transition-all hover:border-(--surface-3) hover:bg-(--surface-1) hover:text-(--ink-1) sm:inline-flex"
				>
					<div class="flex items-center gap-1.5">
						<svg
							class="h-3.5 w-3.5 fill-current transition-transform duration-200 group-hover:scale-110"
							viewBox="0 0 24 24"
						>
							<path
								d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"
							/>
						</svg>
						<span>Star</span>
					</div>
					<span class="h-3 w-px bg-(--surface-2)"></span>
					<span class="flex items-center gap-1 font-mono text-[11px] font-medium text-(--ink-1)">
						<svg class="h-3 w-3 fill-amber-400 text-amber-400" viewBox="0 0 24 24">
							<polygon
								points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"
							/>
						</svg>
						<span>{starCount !== null ? formatStars(starCount) : '—'}</span>
					</span>
				</a>

				<!-- Join Room Action -->
				<a
					href={APP_URL}
					class="hidden h-8 items-center rounded-lg border border-(--surface-2) bg-(--surface-0)/80 px-3 text-xs font-semibold text-(--ink-2) shadow-xs transition-all hover:border-(--surface-3) hover:bg-(--surface-1) hover:text-(--ink-1) sm:inline-flex"
				>
					<span>Join Room</span>
				</a>

				<!-- Quick Create CTA Button -->
				<button
					type="button"
					onclick={createRoom}
					class="group inline-flex h-8 items-center gap-1.5 rounded-lg bg-(--ink-1) px-3.5 text-xs font-bold text-(--surface-0) shadow-sm transition-all hover:bg-(--ink-1)/90 hover:shadow-md"
				>
					<svg
						class="h-3.5 w-3.5 transition-transform duration-200 group-hover:rotate-90"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						stroke-width="2.5"
						stroke-linecap="round"
						stroke-linejoin="round"
					>
						<line x1="12" y1="5" x2="12" y2="19"></line>
						<line x1="5" y1="12" x2="19" y2="12"></line>
					</svg>
					<span>New Room</span>
				</button>
			</div>
		</nav>
	</header>

	<!-- MAIN CONTENT CONTAINER -->
	<main class="mx-auto max-w-6xl px-4 sm:px-6 lg:px-8">
		<!-- HERO SECTION (OpenDesign signature framed title) -->
		<section class="flex flex-col items-center pt-12 pb-14 text-center sm:pt-16 sm:pb-20">
			<!-- HERO TITLE with vector selection bounding box & resize handles -->
			<div
				class="relative mt-8 inline-block max-w-4xl border border-(--accent-lime) bg-(--surface-1)/40 px-6 py-8 sm:px-14 sm:py-12"
			>
				<!-- 4 Corner Vector Resize Handles -->
				<span class="absolute -top-1.5 -left-1.5 h-3 w-3 border border-(--surface-0) bg-(--accent-lime)"></span>
				<span class="absolute -top-1.5 -right-1.5 h-3 w-3 border border-(--surface-0) bg-(--accent-lime)"></span>
				<span class="absolute -bottom-1.5 -left-1.5 h-3 w-3 border border-(--surface-0) bg-(--accent-lime)"></span>
				<span class="absolute -right-1.5 -bottom-1.5 h-3 w-3 border border-(--surface-0) bg-(--accent-lime)"></span>

				<!-- Midpoint Handles for full vector transform box feel -->
				<span class="absolute -top-1.5 left-1/2 -translate-x-1/2 h-3 w-3 border border-(--surface-0) bg-(--accent-lime)"></span>
				<span class="absolute -bottom-1.5 left-1/2 -translate-x-1/2 h-3 w-3 border border-(--surface-0) bg-(--accent-lime)"></span>
				<span class="absolute top-1/2 -left-1.5 -translate-y-1/2 h-3 w-3 border border-(--surface-0) bg-(--accent-lime)"></span>
				<span class="absolute top-1/2 -right-1.5 -translate-y-1/2 h-3 w-3 border border-(--surface-0) bg-(--accent-lime)"></span>

				<p class="font-mono text-xs font-bold tracking-widest text-(--accent-lime) uppercase">
					Real-time Collaborative System
				</p>
				<h1
					class="mt-3 text-3xl font-black tracking-tight text-(--ink-1) sm:text-5xl sm:leading-tight lg:text-6xl"
				>
					Real-time vector whiteboard,<br />
					executed on the edge.
				</h1>
			</div>

			<!-- Hero Subheading -->
					<p class="mt-6 max-w-2xl text-base leading-relaxed text-(--ink-2) sm:text-lg">
						Built for engineers and product teams. Sub-16ms vector input, adaptive 15Hz presence, and
						embedded SQLite persistence. No sign-ups, no tracking cookies, and zero idle compute costs.
					</p>

					<!-- Ticker -->
					<div class="mt-6 flex flex-wrap items-center justify-center gap-4 text-xs font-mono text-(--ink-2)">
						<span class="rounded border border-(--surface-2) px-2 py-1">Latency: &lt;16ms</span>
						<span class="rounded border border-(--surface-2) px-2 py-1">Presence: 15Hz</span>
						<span class="rounded border border-(--surface-2) px-2 py-1">SQLite: LWW</span>
						<span class="rounded border border-(--surface-2) px-2 py-1">Peers: 50</span>
					</div>

			<!-- Hero Action Buttons -->
			<div class="mt-8 flex flex-wrap items-center justify-center gap-3">
				<button
					type="button"
					onclick={createRoom}
					class="rounded-lg bg-(--ink-1) px-6 py-2.5 text-sm font-bold text-(--surface-0) transition-colors hover:bg-(--ink-1)/90"
				>
					Launch Whiteboard
				</button>
				<a
					href={APP_URL}
					class="rounded-lg border border-(--surface-2) bg-(--surface-1) px-5 py-2.5 text-sm font-semibold text-(--ink-1) transition-colors hover:border-(--surface-3)"
				>
					Join Session
				</a>
			</div>
		</section>

		<!-- WORKBENCH CENTERPIECE (Full-Width Workstation Screenshot) -->
		<section id="workbench" class="mb-20 scroll-mt-24">
			<div
				class="overflow-hidden rounded-2xl border border-(--surface-2) bg-(--surface-1) shadow-2xl shadow-black/30"
			>
				<!-- Window Header Bar (macOS traffic lights + URL bar) -->
				<div
					class="flex items-center justify-between border-b border-(--surface-2) bg-(--surface-0)/90 px-4 py-3 sm:px-6"
				>
					<div class="flex items-center gap-2">
						<span class="h-3 w-3 rounded-full bg-[#ef4444]/80"></span>
						<span class="h-3 w-3 rounded-full bg-[#eab308]/80"></span>
						<span class="h-3 w-3 rounded-full bg-[#22c55e]/80"></span>
					</div>
					<div
						class="flex items-center gap-2 rounded-md border border-(--surface-2) bg-(--surface-1) px-3 py-1 font-mono text-xs text-(--ink-2)"
					>
						<svg
							class="h-3.5 w-3.5 text-(--accent-lime)"
							viewBox="0 0 24 24"
							fill="none"
							stroke="currentColor"
							stroke-width="2"
						>
							<rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect>
							<path d="M7 11V7a5 5 0 0 1 10 0v4"></path>
						</svg>
						<span>https://mesh.workers.dev/room/demo-showcase</span>
					</div>
					<div class="w-12"></div>
				</div>

				<!-- Full-Width Screenshot Display -->
				<div class="group relative overflow-hidden bg-black/40">
					<!-- The Screenshot Image Button with zoom lightbox trigger -->
					<button
						type="button"
						onclick={() => (showLightbox = true)}
						class="relative block w-full cursor-zoom-in text-left focus:outline-none"
						title="Click to expand high-resolution screenshot"
					>
						<img
							src="/app-screenshot.png"
							alt="Mesh real-time collaborative vector whiteboard showing distributed cluster architecture, sticky notes, peers, and toolbar"
							class="w-full object-contain transition-transform duration-500 ease-out group-hover:scale-[1.005]"
							loading="eager"
						/>

						<!-- Hover Overlay with Inspect prompt -->
						<div
							class="absolute inset-0 flex items-center justify-center bg-black/30 opacity-0 backdrop-blur-[2px] transition-opacity duration-200 group-hover:opacity-100"
						>
							<div
								class="flex items-center gap-2 rounded-lg border border-(--surface-2) bg-(--surface-1)/95 px-4 py-2 text-xs font-bold text-(--ink-1) shadow-xl"
							>
								<svg
									class="h-4 w-4 text-(--ink-2)"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="2"
								>
									<circle cx="11" cy="11" r="8"></circle>
									<line x1="21" y1="21" x2="16.65" y2="16.65"></line>
									<line x1="11" y1="8" x2="11" y2="14"></line>
									<line x1="8" y1="11" x2="14" y2="11"></line>
								</svg>
								<span>Click to Expand Full Screenshot</span>
							</div>
						</div>
					</button>
				</div>

				<!-- Footer strip under the screenshot -->
				<div
					class="flex flex-wrap items-center justify-between gap-2 border-t border-(--surface-2) bg-(--surface-0)/80 px-4 py-3 text-[11px] text-(--ink-2) sm:px-6"
				>
					<div class="flex items-center gap-2">
						<span class="font-mono text-(--accent-lime)">●</span>
						<span>Authentic vector render loop running at 60fps</span>
					</div>
					<div class="flex items-center gap-4 font-mono text-xs">
						<button
							type="button"
							onclick={() => (showLightbox = true)}
							class="text-(--ink-2) underline underline-offset-4 transition-colors hover:text-(--ink-1)"
						>
							Expand View
						</button>
						<button
							type="button"
							onclick={createRoom}
							class="font-semibold text-(--ink-1) transition-colors hover:text-(--accent-lime)"
						>
							Launch Live Room →
						</button>
					</div>
				</div>
			</div>
		</section>



		<!-- INTERACTIVE ENGINE DOCK (OpenDesign Labs Dock Style) -->
		<section id="engine" class="mb-24 scroll-mt-24">
			<div class="mb-8">
				<div class="inline-flex items-center gap-2">
					<span class="h-px w-5 bg-(--accent-lime)"></span>
					<span class="font-mono text-xs font-bold tracking-widest text-(--accent-lime) uppercase">
						CORE SYSTEMS
					</span>
				</div>
				<h2 class="mt-2 text-2xl font-black tracking-tight text-(--ink-1) sm:text-3xl">
					Engineered for speed, built without bloat.
				</h2>
				<p class="mt-2 text-sm text-(--ink-2)">
					Select a subsystem below to inspect its architecture guarantees and design decisions.
				</p>
			</div>

			<!-- Interactive Subsystem Tabs -->
			<div class="flex flex-wrap items-center gap-2 border-b border-(--surface-2) pb-4">
				<button
					type="button"
					onclick={() => (activeEngineTab = 'vector')}
					class="rounded-lg px-4 py-2 text-xs font-bold transition-all {activeEngineTab ===
					'vector'
						? 'bg-(--ink-1) text-(--surface-0)'
						: 'bg-(--surface-1) text-(--ink-2) hover:text-(--ink-1)'}"
				>
					Vector Engine
				</button>
				<button
					type="button"
					onclick={() => (activeEngineTab = 'storage')}
					class="rounded-lg px-4 py-2 text-xs font-bold transition-all {activeEngineTab ===
					'storage'
						? 'bg-(--ink-1) text-(--surface-0)'
						: 'bg-(--surface-1) text-(--ink-2) hover:text-(--ink-1)'}"
				>
					Durable SQLite
				</button>
				<button
					type="button"
					onclick={() => (activeEngineTab = 'presence')}
					class="rounded-lg px-4 py-2 text-xs font-bold transition-all {activeEngineTab ===
					'presence'
						? 'bg-(--ink-1) text-(--surface-0)'
						: 'bg-(--surface-1) text-(--ink-2) hover:text-(--ink-1)'}"
				>
					Adaptive Presence
				</button>
				<button
					type="button"
					onclick={() => (activeEngineTab = 'security')}
					class="rounded-lg px-4 py-2 text-xs font-bold transition-all {activeEngineTab ===
					'security'
						? 'bg-(--ink-1) text-(--surface-0)'
						: 'bg-(--surface-1) text-(--ink-2) hover:text-(--ink-1)'}"
				>
					Zero-Knowledge Auth
				</button>
			</div>

			<!-- Active Subsystem Card Display -->
			<div class="mt-6 rounded-2xl border border-(--surface-2) bg-(--surface-1) p-6 sm:p-8">
				{#if activeEngineTab === 'vector'}
					<div class="grid grid-cols-1 gap-8 md:grid-cols-2 md:items-center">
						<div>
							<span class="font-mono text-xs font-bold text-(--accent-lime)"
								>// DUAL-LAYER CANVAS</span
							>
							<h3 class="mt-2 text-xl font-black text-(--ink-1)">Sub-16ms Vector Render Loop</h3>
							<p class="mt-3 text-sm leading-relaxed text-(--ink-2)">
								Mesh splits rendering across two decoupled canvas surfaces. The committed buffer
								redraws strictly on shape additions or removals. Active drawing previews, live drag
								bounding boxes, and remote peer cursors execute on an overlay running at a locked
								60fps.
							</p>
							<div class="mt-5 grid grid-cols-2 gap-4 font-mono text-xs">
								<div class="rounded-xl border border-(--surface-2) bg-(--surface-0) p-3">
									<p class="text-(--ink-3)">Smoothing Algorithm</p>
									<p class="mt-1 font-bold text-(--ink-1)">Ramer-Douglas-Peucker</p>
								</div>
								<div class="rounded-xl border border-(--surface-2) bg-(--surface-0) p-3">
									<p class="text-(--ink-3)">Supported Primitives</p>
									<p class="mt-1 font-bold text-(--ink-1)">7 Vector Tools</p>
								</div>
							</div>
						</div>
						<div
							class="rounded-xl border border-(--surface-2) bg-(--surface-0) p-5 font-mono text-xs text-(--ink-2)"
						>
							<div class="text-(--accent-lime)">// Client-Side Geometry Optimization</div>
							<p class="mt-2 text-(--ink-1)">
								const points = ramerDouglasPeucker(rawStroke, 1.2);<br />
								const shape = {'{'} id, type: 'path', data: {'{'} points {'}'}
								{'}'};
							</p>
							<p class="mt-4 text-(--ink-3)">
								/* Geometry computation happens purely in the client worker thread, ensuring DO
								handlers stay &le; 2ms */
							</p>
						</div>
					</div>
				{:else if activeEngineTab === 'storage'}
					<div class="grid grid-cols-1 gap-8 md:grid-cols-2 md:items-center">
						<div>
							<span class="font-mono text-xs font-bold text-emerald-400"
								>// ISOLATE TRANSACTION STORAGE</span
							>
							<h3 class="mt-2 text-xl font-black text-(--ink-1)">Deterministic Monotonic LWW</h3>
							<p class="mt-3 text-sm leading-relaxed text-(--ink-2)">
								Each whiteboard room operates as an independent Cloudflare Durable Object isolate
								with its own embedded SQLite database. Multi-user concurrent writes resolve via
								Last-Write-Wins with clock-skew safeguards, preventing conflicting mutations without
								centralized database bottlenecks.
							</p>
							<div class="mt-5 grid grid-cols-2 gap-4 font-mono text-xs">
								<div class="rounded-xl border border-(--surface-2) bg-(--surface-0) p-3">
									<p class="text-(--ink-3)">Storage Medium</p>
									<p class="mt-1 font-bold text-(--ink-1)">Embedded SQLite</p>
								</div>
								<div class="rounded-xl border border-(--surface-2) bg-(--surface-0) p-3">
									<p class="text-(--ink-3)">Clock-Skew Cap</p>
									<p class="mt-1 font-bold text-(--ink-1)">+5000ms Clamping</p>
								</div>
							</div>
						</div>
						<div
							class="rounded-xl border border-(--surface-2) bg-(--surface-0) p-5 font-mono text-xs text-(--ink-2)"
						>
							<div class="text-emerald-400">-- Embedded Transactional Upsert</div>
							<p class="mt-2 text-(--ink-1)">
								INSERT INTO shapes (id, type, x, y, updated_at)<br />
								VALUES (?, ?, ?, ?, ?)<br />
								ON CONFLICT(id) DO UPDATE SET<br />
								&nbsp;&nbsp;x = excluded.x, updated_at = excluded.updated_at<br />
								WHERE excluded.updated_at &gt;= shapes.updated_at;
							</p>
						</div>
					</div>
				{:else if activeEngineTab === 'presence'}
					<div class="grid grid-cols-1 gap-8 md:grid-cols-2 md:items-center">
						<div>
							<span class="font-mono text-xs font-bold text-cyan-400">// WEBSOCKET HIBERNATION</span
							>
							<h3 class="mt-2 text-xl font-black text-(--ink-1)">Adaptive 15Hz Presence</h3>
							<p class="mt-3 text-sm leading-relaxed text-(--ink-2)">
								Cursor tracking is broadcast in memory and never touches disk. While alone in a
								room, cursor transmissions are completely suppressed, saving over 100,000
								invocations per hour and preserving Cloudflare Free Tier quotas. When collaborating,
								transmissions adaptively stream at 15Hz with deadband filtering.
							</p>
							<div class="mt-5 grid grid-cols-2 gap-4 font-mono text-xs">
								<div class="rounded-xl border border-(--surface-2) bg-(--surface-0) p-3">
									<p class="text-(--ink-3)">Solo Quota Usage</p>
									<p class="mt-1 font-bold text-(--ink-1)">0 Cursor Requests</p>
								</div>
								<div class="rounded-xl border border-(--surface-2) bg-(--surface-0) p-3">
									<p class="text-(--ink-3)">Deadband Filter</p>
									<p class="mt-1 font-bold text-(--ink-1)">&lt; 2px Movement</p>
								</div>
							</div>
						</div>
						<div
							class="rounded-xl border border-(--surface-2) bg-(--surface-0) p-5 font-mono text-xs text-(--ink-2)"
						>
							<div class="text-cyan-400">// In-Memory Presence Attachment</div>
							<p class="mt-2 text-(--ink-1)">
								if (this.peers.length === 0 &amp;&amp; cursor !== null) return;<br />
								if (distSq &lt; 4 &amp;&amp; selectionUnchanged) return;<br />
								ws.serializeAttachment({'{'} userId, name, color, cursor {'}'});
							</p>
							<p class="mt-4 text-(--ink-3)">
								/* Ephemeral packets bypass disk writes, eliminating lock contention */
							</p>
						</div>
					</div>
				{:else if activeEngineTab === 'security'}
					<div class="grid grid-cols-1 gap-8 md:grid-cols-2 md:items-center">
						<div>
							<span class="font-mono text-xs font-bold text-purple-400"
								>// CRYPTOGRAPHIC VERIFICATION</span
							>
							<h3 class="mt-2 text-xl font-black text-(--ink-1)">Zero-Knowledge Room Locks</h3>
							<p class="mt-3 text-sm leading-relaxed text-(--ink-2)">
								Password-protected rooms enforce client-side derivation with PBKDF2-SHA256 (100,000
								iterations). Unauthenticated sockets are strictly withheld from receiving shapes,
								presence streams, or mutation events until authenticated. Brute-force throttling and
								connection timeouts prevent automated attacks.
							</p>
							<div class="mt-5 grid grid-cols-2 gap-4 font-mono text-xs">
								<div class="rounded-xl border border-(--surface-2) bg-(--surface-0) p-3">
									<p class="text-(--ink-3)">Derivation Iterations</p>
									<p class="mt-1 font-bold text-(--ink-1)">100,000 Rounds</p>
								</div>
								<div class="rounded-xl border border-(--surface-2) bg-(--surface-0) p-3">
									<p class="text-(--ink-3)">Auth Gate Policy</p>
									<p class="mt-1 font-bold text-(--ink-1)">Strict Isolation</p>
								</div>
							</div>
						</div>
						<div
							class="rounded-xl border border-(--surface-2) bg-(--surface-0) p-5 font-mono text-xs text-(--ink-2)"
						>
							<div class="text-purple-400">// Constant-Time Hash Matching</div>
							<p class="mt-2 text-(--ink-1)">
								const actual = await this.hashPassword(input, salt);<br />
								return this.hashesEqual(actual, expectedHash);
							</p>
							<p class="mt-4 text-(--ink-3)">
								/* Rate limits: 5-second lockout after 5 fails; disconnect code 1008 after 10
								attempts */
							</p>
						</div>
					</div>
				{/if}
			</div>
		</section>

		<!-- 3-TIER ARCHITECTURE VISUALIZER -->
		<section id="architecture" class="mb-24 scroll-mt-24">
			<div class="mb-10 text-center sm:text-left">
				<div class="inline-flex items-center gap-2">
					<span class="h-px w-5 bg-(--accent-lime)"></span>
					<span class="font-mono text-xs font-bold tracking-widest text-(--accent-lime) uppercase">
						SYSTEM TOPOLOGY
					</span>
				</div>
				<h2 class="mt-2 text-2xl font-black tracking-tight text-(--ink-1) sm:text-3xl">
					Single origin. Zero external database dependencies.
				</h2>
			</div>

			<div class="grid grid-cols-1 gap-6 md:grid-cols-3">
				<!-- Tier 1: Client Layer -->
				<div class="rounded-2xl border border-(--surface-2) bg-(--surface-1) p-6">
					<div
						class="flex h-10 w-10 items-center justify-center rounded-xl bg-indigo-500/10 text-indigo-400"
					>
						<span class="font-mono text-xs font-bold">01</span>
					</div>
					<h3 class="mt-4 text-base font-bold text-(--ink-1)">Browser Client</h3>
					<p class="mt-2 text-xs leading-relaxed text-(--ink-2)">
						Runs Svelte 5 with fine-grained reactivity ($state/$effect). Captures pointer events,
						smooths strokes with RDP, and renders optimistic updates locally.
					</p>
				</div>

				<!-- Tier 2: Edge Router -->
				<div class="rounded-2xl border border-(--surface-2) bg-(--surface-1) p-6">
					<div
						class="flex h-10 w-10 items-center justify-center rounded-xl bg-cyan-500/10 text-cyan-400"
					>
						<span class="font-mono text-xs font-bold">02</span>
					</div>
					<h3 class="mt-4 text-base font-bold text-(--ink-1)">Anycast Edge Worker</h3>
					<p class="mt-2 text-xs leading-relaxed text-(--ink-2)">
						hooks.server.ts intercepts /api/room/:id/ws before page resolution, routing the raw
						upgrade to the dedicated room isolate via idFromName(roomId).
					</p>
				</div>

				<!-- Tier 3: Room Isolate & SQLite -->
				<div class="rounded-2xl border border-(--surface-2) bg-(--surface-1) p-6">
					<div
						class="flex h-10 w-10 items-center justify-center rounded-xl bg-emerald-500/10 text-emerald-400"
					>
						<span class="font-mono text-xs font-bold">03</span>
					</div>
					<h3 class="mt-4 text-base font-bold text-(--ink-1)">Durable Object & SQLite</h3>
					<p class="mt-2 text-xs leading-relaxed text-(--ink-2)">
						One isolate per room. Accepts hibernating WebSocket connections, resolves LWW conflicts,
						and commits to single-tenant transactional SQLite storage.
					</p>
				</div>
			</div>
		</section>

		<!-- SELF-HOSTING TERMINAL STATION -->
		<section id="self-host" class="mb-24 scroll-mt-24">
			<div class="mb-8">
				<div class="inline-flex items-center gap-2">
					<span class="h-px w-5 bg-(--accent-lime)"></span>
					<span class="font-mono text-xs font-bold tracking-widest text-(--accent-lime) uppercase">
						DEPLOYMENT
					</span>
				</div>
				<h2 class="mt-2 text-2xl font-black tracking-tight text-(--ink-1) sm:text-3xl">
					Self-host in your own cloud or container.
				</h2>
			</div>

			<div
				class="overflow-hidden rounded-2xl border border-(--surface-2) bg-(--surface-1) shadow-xl"
			>
				<!-- Terminal Tab Bar -->
				<div
					class="flex flex-wrap items-center justify-between gap-4 border-b border-(--surface-2) bg-(--surface-0)/80 px-4 py-3"
				>
					<div class="flex items-center gap-2">
						<button
							type="button"
							onclick={() => (activeSelfHostTab = 'docker')}
							class="rounded-lg px-3 py-1.5 font-mono text-xs font-semibold transition-colors {activeSelfHostTab ===
							'docker'
								? 'bg-(--surface-2) text-(--ink-1)'
								: 'text-(--ink-3) hover:text-(--ink-1)'}"
						>
							Docker Compose
						</button>
						<button
							type="button"
							onclick={() => (activeSelfHostTab = 'bun')}
							class="rounded-lg px-3 py-1.5 font-mono text-xs font-semibold transition-colors {activeSelfHostTab ===
							'bun'
								? 'bg-(--surface-2) text-(--ink-1)'
								: 'text-(--ink-3) hover:text-(--ink-1)'}"
						>
							Bun Runtime
						</button>
						<button
							type="button"
							onclick={() => (activeSelfHostTab = 'deploy')}
							class="rounded-lg px-3 py-1.5 font-mono text-xs font-semibold transition-colors {activeSelfHostTab ===
							'deploy'
								? 'bg-(--surface-2) text-(--ink-1)'
								: 'text-(--ink-3) hover:text-(--ink-1)'}"
						>
							Cloudflare Deploy
						</button>
					</div>

					<button
						type="button"
						onclick={() => copyCode(snippets[activeSelfHostTab])}
						class="flex items-center gap-1.5 rounded-lg border border-(--surface-2) bg-(--surface-1) px-3 py-1 text-xs font-semibold text-(--ink-2) transition-colors hover:border-(--surface-3) hover:text-(--ink-1)"
					>
						{#if copied}
							<span class="text-(--accent-lime)">✓ Copied!</span>
						{:else}
							<span>Copy Command</span>
						{/if}
					</button>
				</div>

				<!-- Terminal Code View -->
				<div class="p-6 font-mono text-xs leading-relaxed sm:text-sm">
					<pre class="overflow-x-auto text-(--ink-1)"><code>{snippets[activeSelfHostTab]}</code
						></pre>
				</div>
			</div>
		</section>

		<!-- PROTOCOL SPECS & TECHNICAL LIMITS TABLE -->
		<section id="tech-specs" class="mb-24 scroll-mt-24">
			<div class="mb-8">
				<div class="inline-flex items-center gap-2">
					<span class="h-px w-5 bg-(--accent-lime)"></span>
					<span class="font-mono text-xs font-bold tracking-widest text-(--accent-lime) uppercase">
						SPECIFICATION
					</span>
				</div>
				<h2 class="mt-2 text-2xl font-black tracking-tight text-(--ink-1) sm:text-3xl">
					Hard protocol bounds &amp; invariants.
				</h2>
			</div>

			<div class="overflow-hidden rounded-2xl border border-(--surface-2) bg-(--surface-1)">
				<div class="divide-y divide-(--surface-2) text-xs">
					<div class="flex flex-col justify-between px-6 py-4 sm:flex-row sm:items-center">
						<span class="font-semibold text-(--ink-1)">Max Concurrent Connections</span>
						<span class="font-mono text-(--ink-2)">50 peers per room (with unauthed eviction)</span>
					</div>
					<div class="flex flex-col justify-between px-6 py-4 sm:flex-row sm:items-center">
						<span class="font-semibold text-(--ink-1)">Max Shapes Per Room</span>
						<span class="font-mono text-(--ink-2)">10,000 vector shapes</span>
					</div>
					<div class="flex flex-col justify-between px-6 py-4 sm:flex-row sm:items-center">
						<span class="font-semibold text-(--ink-1)">Rate Limiting</span>
						<span class="font-mono text-(--ink-2)">150 messages/sec sliding window per socket</span>
					</div>
					<div class="flex flex-col justify-between px-6 py-4 sm:flex-row sm:items-center">
						<span class="font-semibold text-(--ink-1)">Payload Frame Ceiling</span>
						<span class="font-mono text-(--ink-2)">64 KB (rejects frame with code 1009)</span>
					</div>
					<div class="flex flex-col justify-between px-6 py-4 sm:flex-row sm:items-center">
						<span class="font-semibold text-(--ink-1)">Conflict Resolution</span>
						<span class="font-mono text-(--ink-2)"
							>Monotonic Last-Write-Wins (LWW) with clock skew cap</span
						>
					</div>
					<div class="flex flex-col justify-between px-6 py-4 sm:flex-row sm:items-center">
						<span class="font-semibold text-(--ink-1)">Presence Broadcast</span>
						<span class="font-mono text-(--ink-2)"
							>Adaptive 15Hz with solo silence and deadband filtering</span
						>
					</div>
					<div class="flex flex-col justify-between px-6 py-4 sm:flex-row sm:items-center">
						<span class="font-semibold text-(--ink-1)">Authentication Hash</span>
						<span class="font-mono text-(--ink-2)"
							>PBKDF2-SHA256 (100,000 iterations via crypto.subtle)</span
						>
					</div>
				</div>
			</div>
		</section>
	</main>

	<!-- EDITORIAL TYPOGRAPHIC FOOTER -->
	<footer class="border-t border-(--surface-2) bg-(--surface-0) py-12 transition-colors">
		<div
			class="mx-auto flex max-w-6xl flex-col items-center justify-between gap-6 px-4 sm:flex-row sm:px-6 lg:px-8"
		>
			<div class="flex items-center gap-3">
				<img src={logo} alt="Mesh Logo" class="h-6 w-6 rounded-md object-contain" />
				<span class="text-sm font-bold text-(--ink-1)">Mesh</span>
				<span class="text-xs text-(--ink-3)">// MIT Open Source License</span>
			</div>

			<div class="flex items-center gap-6 text-xs text-(--ink-2)">
				<a
					href="https://github.com/{REPO}"
					target="_blank"
					rel="noreferrer"
					class="transition-colors hover:text-(--ink-1)"
				>
					GitHub Repository
				</a>
				<a
					href="https://github.com/{REPO}/blob/main/LICENSE"
					target="_blank"
					rel="noreferrer"
					class="transition-colors hover:text-(--ink-1)"
				>
					License
				</a>
				<a
					href="https://github.com/{REPO}/releases"
					target="_blank"
					rel="noreferrer"
					class="transition-colors hover:text-(--ink-1)"
				>
					Releases
				</a>
			</div>
		</div>
	</footer>

	<!-- LIGHTBOX SCREENSHOT MODAL -->
	{#if showLightbox}
		<div
			class="fixed inset-0 z-50 flex items-center justify-center bg-black/80 p-3 backdrop-blur-md sm:p-6"
			role="dialog"
			aria-modal="true"
			aria-label="High resolution whiteboard screenshot"
		>
			<!-- Backdrop Click Area -->
			<button
				type="button"
				class="absolute inset-0 h-full w-full cursor-default bg-transparent"
				onclick={() => (showLightbox = false)}
				aria-label="Dismiss modal"
			></button>

			<div
				class="relative z-10 flex max-h-[94vh] w-full max-w-6xl flex-col overflow-hidden rounded-2xl border border-(--surface-2) bg-(--surface-1) shadow-2xl"
			>
				<!-- Modal Header Bar -->
				<div
					class="flex items-center justify-between border-b border-(--surface-2) bg-(--surface-0) px-4 py-3"
				>
					<div class="flex items-center gap-2.5">
						<span class="h-3 w-3 rounded-full bg-[#ef4444]/80"></span>
						<span class="h-3 w-3 rounded-full bg-[#eab308]/80"></span>
						<span class="h-3 w-3 rounded-full bg-[#22c55e]/80"></span>
						<span class="ml-2 font-mono text-xs font-semibold text-(--ink-1)">
							Mesh Production Canvas
						</span>
					</div>

					<div class="flex items-center gap-3">
						<button
							type="button"
							onclick={createRoom}
							class="rounded-lg bg-(--ink-1) px-3 py-1 text-xs font-bold text-(--surface-0) transition-colors hover:bg-(--ink-1)/90"
						>
							Launch Live Room
						</button>
						<button
							type="button"
							onclick={() => (showLightbox = false)}
							class="flex h-7 w-7 items-center justify-center rounded-lg border border-(--surface-2) bg-(--surface-1) text-xs text-(--ink-2) transition-colors hover:bg-(--surface-2) hover:text-(--ink-1)"
							aria-label="Close screenshot preview"
						>
							✕
						</button>
					</div>
				</div>

				<!-- Image Display -->
				<div class="overflow-auto bg-black/40 p-2 sm:p-4">
					<img
						src="/app-screenshot.png"
						alt="Mesh live collaborative vector canvas showing architecture diagram, sticky notes, cursors, and minimap"
						class="w-full rounded-lg object-contain shadow-md"
					/>
				</div>

				<!-- Modal Footer -->
				<div
					class="flex items-center justify-between border-t border-(--surface-2) bg-(--surface-0) px-4 py-2.5 text-[11px] text-(--ink-2)"
				>
					<div class="flex items-center gap-2">
						<span class="h-1.5 w-1.5 rounded-full bg-(--accent-lime)"></span>
						<span
							>Authentic live canvas with Dual-layer 60fps rendering, SQLite storage, and 15Hz
							presence</span
						>
					</div>
					<span class="hidden font-mono text-[10px] text-(--ink-3) sm:inline-block"
						>Press ESC or click outside to close</span
					>
				</div>
			</div>
		</div>
	{/if}
</div>
