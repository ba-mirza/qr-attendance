<script lang="ts">
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
	import { supabase } from '$lib/supabase';

	let status = 'Проверка авторизации...';

	onMount(async () => {
		const {
			data: { user }
		} = await supabase.auth.getUser();

		if (!user) {
			status = 'Ошибка авторизации';
			setTimeout(() => goto('/auth'), 2000);
			return;
		}

		const { data: owner } = await supabase
			.from('owners')
			.select('organization_id')
			.eq('auth_user_id', user.id)
			.single();

		if (owner) {
			goto('/dashboard');
		} else {
			goto('/onboarding');
		}
	});
</script>

<div class="flex min-h-screen items-center justify-center bg-gray-50">
	<div class="text-center">
		<div class="mx-auto mb-4 h-12 w-12 animate-spin rounded-full border-b-2 border-blue-600"></div>
		<p class="text-gray-600">{status}</p>
	</div>
</div>
