<script>
	import { onMount } from 'svelte';

	let posts = $state([]);
	let loadingInitial = $state(true);
	let loadingMore = $state(false);
	let error = $state(null);
	let selectedPost = $state(null);
	let comments = $state([]);
	let loadingComments = $state(false);

	let page = $state(1);
	const limit = 9;
	let hasMore = $state(true);

	async function loadPosts(pageNum) {
		if (loadingMore || (!hasMore && pageNum !== 1)) return;

		try {
			if (pageNum === 1) {
				loadingInitial = true;
			} else {
				loadingMore = true;
			}
			error = null;

			const res = await fetch(`https://jsonplaceholder.typicode.com/posts?_page=${pageNum}&_limit=${limit}`);
			if (!res.ok) throw new Error('記事の取得に失敗しました');

			const newPosts = await res.json();

			if (newPosts.length === 0) {
				hasMore = false;
			} else {
				posts = [...posts, ...newPosts];
				page = pageNum;
				if (posts.length >= 100 || newPosts.length < limit) {
					hasMore = false;
				}
			}
		} catch (err) {
			error = err.message;
		} finally {
			loadingInitial = false;
			loadingMore = false;
		}
	}

	// Svelteアクション: 要素がDOMに現れたタイミングで監視を開始する
	function infiniteScroll(node) {
		const observer = new IntersectionObserver(
			(entries) => {
				const first = entries[0];
				if (first.isIntersecting && hasMore && !loadingInitial && !loadingMore) {
					loadPosts(page + 1);
				}
			},
			{ rootMargin: '200px' }
		);

		observer.observe(node);

		return {
			destroy() {
				observer.disconnect();
			}
		};
	}

	async function openPost(post) {
		selectedPost = post;
		loadingComments = true;
		try {
			const res = await fetch(`https://jsonplaceholder.typicode.com/posts/${post.id}/comments`);
			comments = await res.json();
		} catch (err) {
			comments = [];
		} finally {
			loadingComments = false;
		}
	}

	function closeModal() {
		selectedPost = null;
		comments = [];
	}

	onMount(() => {
		loadPosts(1);
	});
</script>

<div class="flex items-center justify-between mb-8 pb-4 border-b border-neutral-800/80">
	<div>
		<h1 class="text-3xl font-extrabold text-white tracking-tight">Feed</h1>
		<p class="text-xs text-neutral-400 mt-1">
			スクロールすると次の記事を自動ロードします ({posts.length} 件ロード済み)
		</p>
	</div>
	
	{#if loadingMore}
		<div class="flex items-center gap-2 text-xs font-mono text-cyan-400 bg-cyan-950/40 border border-cyan-800/40 px-3 py-1.5 rounded-lg animate-pulse">
			<span class="w-2 h-2 rounded-full bg-cyan-400"></span>
			Loading more...
		</div>
	{/if}
</div>

{#if loadingInitial}
	<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
		{#each Array(limit) as _}
			<div class="h-60 rounded-2xl bg-neutral-900/60 border border-neutral-800/80 animate-pulse p-6 flex flex-col justify-between">
				<div>
					<div class="h-4 w-16 bg-neutral-800 rounded mb-4"></div>
					<div class="h-6 w-3/4 bg-neutral-800 rounded mb-3"></div>
					<div class="h-3 w-full bg-neutral-800 rounded mb-2"></div>
					<div class="h-3 w-4/5 bg-neutral-800 rounded"></div>
				</div>
				<div class="h-4 w-20 bg-neutral-800 rounded"></div>
			</div>
		{/each}
	</div>
{:else if error && posts.length === 0}
	<div class="p-8 rounded-2xl bg-rose-950/20 border border-rose-900/40 text-rose-300 text-center">
		<p class="font-medium">{error}</p>
		<button 
			onclick={() => loadPosts(1)} 
			class="mt-4 px-4 py-2 bg-rose-500/20 hover:bg-rose-500/30 text-rose-200 rounded-xl text-sm transition"
		>
			再試行する
		</button>
	</div>
{:else}
	<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
		{#each posts as post (post.id)}
			<article
				class="group relative flex flex-col justify-between p-6 rounded-2xl bg-neutral-900/40 border border-neutral-800/80 hover:border-cyan-500/40 hover:bg-neutral-900/80 transition-all duration-300 hover:-translate-y-1 hover:shadow-[0_10px_25px_-5px_rgba(0,0,0,0.5)]"
			>
				<div>
					<span class="inline-block text-[11px] font-mono font-medium text-cyan-400 bg-cyan-950/50 border border-cyan-800/40 px-2.5 py-0.5 rounded-md mb-3">
						POST #{post.id}
					</span>
					<h2 class="text-base font-semibold text-neutral-100 group-hover:text-cyan-300 transition-colors line-clamp-2 mb-2 leading-snug">
						{post.title}
					</h2>
					<p class="text-xs text-neutral-400 line-clamp-3 leading-relaxed">
						{post.body}
					</p>
				</div>

				<button
					onclick={() => openPost(post)}
					class="mt-6 flex items-center justify-between text-xs font-medium text-neutral-400 hover:text-white transition-colors group/btn pt-4 border-t border-neutral-800/60"
				>
					<span>続きを読む</span>
					<span class="transform transition-transform group-hover/btn:translate-x-1 text-cyan-400">→</span>
				</button>
			</article>
		{/each}
	</div>

	<!-- use:infiniteScroll でマウント時に確実に監視をスタート -->
	<div use:infiniteScroll class="py-12 flex justify-center items-center min-h-[80px]">
		{#if loadingMore}
			<div class="flex items-center gap-3 text-neutral-400 text-sm">
				<div class="w-4 h-4 border-2 border-cyan-400 border-t-transparent rounded-full animate-spin"></div>
				<span>新しい記事を読み込み中...</span>
			</div>
		{:else if !hasMore}
			<div class="text-center">
				<div class="inline-block w-8 h-[1px] bg-neutral-800 mb-2"></div>
				<p class="text-xs text-neutral-500 font-mono tracking-wider">ALL POSTS LOADED (100 / 100)</p>
			</div>
		{/if}
	</div>
{/if}

<!-- 詳細モーダル -->
{#if selectedPost}
	<div class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/70 backdrop-blur-sm animate-fade-in">
		<div class="relative w-full max-w-2xl max-h-[85vh] overflow-y-auto bg-neutral-900 border border-neutral-800 rounded-2xl p-6 md:p-8 shadow-2xl space-y-6">
			<button
				onclick={closeModal}
				class="absolute top-6 right-6 text-neutral-400 hover:text-white text-lg transition-colors"
			>
				✕
			</button>

			<div>
				<span class="text-xs font-mono text-cyan-400">POST #{selectedPost.id}</span>
				<h2 class="text-xl md:text-2xl font-bold text-white mt-1 leading-snug">
					{selectedPost.title}
				</h2>
			</div>

			<p class="text-sm text-neutral-300 leading-relaxed bg-neutral-950/40 p-4 rounded-xl border border-neutral-800/40">
				{selectedPost.body}
			</p>

			<div class="pt-4 border-t border-neutral-800">
				<h3 class="text-sm font-semibold text-neutral-200 mb-4 flex items-center gap-2">
					Comments 
					{#if !loadingComments}
						<span class="text-xs text-neutral-500 font-normal">({comments.length})</span>
					{/if}
				</h3>

				{#if loadingComments}
					<div class="space-y-3">
						<div class="h-14 bg-neutral-800/50 rounded-xl animate-pulse"></div>
						<div class="h-14 bg-neutral-800/50 rounded-xl animate-pulse"></div>
					</div>
				{:else}
					<div class="space-y-3 max-h-60 overflow-y-auto pr-1">
						{#each comments as comment}
							<div class="p-3 bg-neutral-950/60 rounded-xl border border-neutral-800/40 text-xs">
								<div class="font-medium text-cyan-300/90 truncate">{comment.email}</div>
								<p class="text-neutral-400 mt-1 leading-relaxed">{comment.body}</p>
							</div>
						{/each}
					</div>
				{/if}
			</div>
		</div>
	</div>
{/if}