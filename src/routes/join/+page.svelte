<script lang="ts">
	import logo from '$lib/assets/logo.png';

	const APP_URL = import.meta.env.VITE_APP_URL || 'https://app.mesh.asy.web.id';

	let joinInput = $state('');
	let joinPassword = $state('');
	let joinError = $state('');

	function handleJoin(e: Event) {
		e.preventDefault();
		joinError = '';
		let trimmed = joinInput.trim();
		if (!trimmed) {
			joinError = 'Please enter a room code or link';
			return;
		}

		if (trimmed.includes('/room/')) {
			trimmed = trimmed.split('/room/').pop()?.split(/[?#/]/)[0] || trimmed;
		}

		if (!/^[a-zA-Z0-9_-]{3,64}$/.test(trimmed)) {
			joinError = 'Room ID must be 3-64 alphanumeric characters, underscores, or hyphens';
			return;
		}

		if (joinPassword) {
			try {
				sessionStorage.setItem(`mesh_new_room_pw_${trimmed}`, joinPassword);
			} catch {
				// Ignore storage errors
			}
		}

		if (typeof window !== 'undefined') {
			window.location.href = `${APP_URL}/room/${trimmed}`;
		}
	}

	function generateRoomId(): string {
		const chars = 'abcdefghjkmnpqrstuvwxyz23456789';
		let result = '';
		for (let i = 0; i < 6; i++) {
			result += chars.charAt(Math.floor(Math.random() * chars.length));
		}
		return `room-${result}`;
	}

	function createNewRoom() {
		const newId = generateRoomId();
		if (typeof window !== 'undefined') {
			window.location.href = `${APP_URL}/room/${newId}`;
		}
	}
</script>

<svelte:head>
	<title>Join Room // Mesh</title>
</svelte:head>

<div
	class="flex min-h-screen w-screen flex-col items-center justify-center bg-(--surface-0) p-4 text-(--ink-1)"
>
	<!-- Top Navigation Back Link -->
	<header class="absolute top-6 left-6 sm:top-8 sm:left-8">
		<a
			href="/"
			class="flex items-center gap-2 font-mono text-xs font-semibold text-(--ink-2) transition-colors hover:text-(--ink-1)"
		>
			<span>←</span>
			<span>Back to Mesh</span>
		</a>
	</header>

	<main class="w-full max-w-md">
		<div class="rounded-2xl border border-(--surface-2) bg-(--surface-1) p-6 shadow-xl sm:p-8">
			<div class="flex items-center gap-3 border-b border-(--surface-2) pb-5">
				<img src={logo} alt="Mesh Logo" class="h-8 w-8 rounded-lg object-contain" />
				<div>
					<h1 class="text-lg font-bold text-(--ink-1)">Join Whiteboard</h1>
					<p class="text-xs text-(--ink-2)">Enter an existing room ID or paste a shared link</p>
				</div>
			</div>

			<form onsubmit={handleJoin} class="mt-6 space-y-4">
				<div>
					<label for="room-id" class="block text-xs font-semibold text-(--ink-2)">
						Room Code or Link
					</label>
					<input
						id="room-id"
						type="text"
						bind:value={joinInput}
						placeholder="e.g. room-qk5qt3 or full URL"
						autocomplete="off"
						class="mt-1.5 w-full rounded-lg border border-(--surface-2) bg-(--surface-0) px-3.5 py-2.5 font-mono text-xs text-(--ink-1) placeholder-(--ink-3) transition-colors focus:border-(--accent-lime) focus:outline-none"
					/>
				</div>

				<div>
					<label for="room-password" class="block text-xs font-semibold text-(--ink-2)">
						Room Password <span class="font-normal text-(--ink-3)">(Optional)</span>
					</label>
					<input
						id="room-password"
						type="password"
						bind:value={joinPassword}
						placeholder="Leave empty if room is public"
						autocomplete="current-password"
						class="mt-1.5 w-full rounded-lg border border-(--surface-2) bg-(--surface-0) px-3.5 py-2.5 text-xs text-(--ink-1) placeholder-(--ink-3) transition-colors focus:border-(--accent-lime) focus:outline-none"
					/>
				</div>

				{#if joinError}
					<p class="text-xs text-red-400">{joinError}</p>
				{/if}

				<button
					type="submit"
					class="w-full rounded-lg bg-(--ink-1) py-2.5 text-xs font-bold text-(--surface-0) transition-opacity hover:opacity-90"
				>
					Join Session
				</button>
			</form>

			<div
				class="mt-6 flex items-center justify-between border-t border-(--surface-2) pt-4 text-xs text-(--ink-2)"
			>
				<span>Need a new whiteboard?</span>
				<button
					type="button"
					onclick={createNewRoom}
					class="font-semibold text-(--ink-1) underline underline-offset-4 hover:text-(--accent-lime)"
				>
					Create New Room
				</button>
			</div>
		</div>
	</main>
</div>
