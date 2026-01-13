<script lang="ts">
	import { supabase } from '$lib/supabase';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
	import { z } from 'zod';

	let user: any = null;
	let loading = false;
	let error = '';

	const schema = z.object({
		name: z.string().min(2, 'Минимум 2 символа'),
		industry: z.string().min(1, 'Выберите тип'),
		employee_count: z.number().min(1, 'Укажите количество')
	});

	let formData = {
		name: '',
		industry: '',
		employee_count: 10
	};

	onMount(async () => {
		const {
			data: { user: currentUser }
		} = await supabase.auth.getUser();
		if (!currentUser) {
			goto('/auth');
			return;
		}
		user = currentUser;
	});

	async function handleSubmit() {
		loading = true;
		error = '';

		try {
			schema.parse(formData);

			const slug = formData.name
				.toLowerCase()
				.replace(/[^a-zа-яё0-9]+/g, '-')
				.replace(/^-+|-+$/g, '');

			const { data: org, error: orgError } = await supabase
				.from('organizations')
				.insert({
					name: formData.name,
					slug: slug + '-' + Date.now(),
					industry: formData.industry,
					employee_count: formData.employee_count
				})
				.select()
				.single();

			if (orgError) throw orgError;

			const { error: ownerError } = await supabase.from('owners').insert({
				organization_id: org.id,
				auth_user_id: user.id,
				email: user.email,
				full_name: user.user_metadata?.full_name || user.email?.split('@')[0] || 'Владелец'
			});

			if (ownerError) throw ownerError;

			goto('/');
		} catch (err: any) {
			if (err instanceof z.ZodError) {
				error = err.errors[0].message;
			} else {
				error = err.message || 'Произошла ошибка';
			}
		} finally {
			loading = false;
		}
	}
</script>

<div class="flex min-h-screen items-center justify-center bg-gray-50 px-4">
	<div class="w-full max-w-md">
		<div class="mb-8 text-center">
			<h1 class="mb-2 text-3xl font-bold text-gray-900">Создать организацию</h1>
			<p class="text-gray-600">Заполните данные о вашей компании</p>
		</div>

		<div class="rounded-lg bg-white p-8 shadow-md">
			<form on:submit|preventDefault={handleSubmit} class="space-y-4">
				<div>
					<label for="name" class="mb-1 block text-sm font-medium text-gray-700">
						Наименование ТОО *
					</label>
					<input
						id="name"
						type="text"
						bind:value={formData.name}
						placeholder="ТОО Компания"
						class="w-full rounded-lg border border-gray-300 px-4 py-2 focus:border-transparent focus:ring-2 focus:ring-blue-500"
						required
					/>
				</div>

				<div>
					<label for="industry" class="mb-1 block text-sm font-medium text-gray-700">
						Тип компании *
					</label>
					<select
						id="industry"
						bind:value={formData.industry}
						class="w-full rounded-lg border border-gray-300 px-4 py-2 focus:border-transparent focus:ring-2 focus:ring-blue-500"
						required
					>
						<option value="">Выберите тип</option>
						<option value="IT">IT / Технологии</option>
						<option value="Продажа">Продажа / Ритейл</option>
						<option value="Завод">Завод / Производство</option>
						<option value="Строительство">Строительство</option>
						<option value="Логистика">Логистика</option>
						<option value="Образование">Образование</option>
						<option value="Другое">Другое</option>
					</select>
				</div>

				<div>
					<label for="count" class="mb-1 block text-sm font-medium text-gray-700">
						Количество сотрудников *
					</label>
					<input
						id="count"
						type="number"
						bind:value={formData.employee_count}
						min="1"
						class="w-full rounded-lg border border-gray-300 px-4 py-2 focus:border-transparent focus:ring-2 focus:ring-blue-500"
						required
					/>
				</div>

				{#if error}
					<div class="rounded-lg bg-red-50 p-3 text-sm text-red-600">
						{error}
					</div>
				{/if}

				<button
					type="submit"
					disabled={loading}
					class="w-full rounded-lg bg-blue-600 py-2 font-semibold text-white hover:bg-blue-700 disabled:opacity-50"
				>
					{loading ? 'Создаём...' : 'Создать организацию'}
				</button>
			</form>
		</div>
	</div>
</div>
