<script lang="ts">
	import { createEventDispatcher } from "svelte";
	import { base } from "$app/paths";
	import { goto } from "$app/navigation";
	import type { Model } from "$lib/types/Model";
	import type { Assistant } from "$lib/types/Assistant";
	import { useSettingsStore } from "$lib/stores/settings";
	import { formatUserCount } from "$lib/utils/formatUserCount";
	import IconGear from "~icons/bi/gear-fill";
	import IconInternet from "../icons/IconInternet.svelte";
	import CarbonExport from "~icons/carbon/export";
	import CarbonCheckmark from "~icons/carbon/checkmark";
	import CarbonRenew from "~icons/carbon/renew";
	import CarbonUserMultiple from "~icons/carbon/user-multiple";
	import CarbonTools from "~icons/carbon/tools";

	import { share } from "$lib/utils/share";
	import { publicConfig } from "$lib/utils/PublicConfig.svelte";

	import { page } from "$app/state";
	import { useTranslations } from "$lib/stores/translations";

	interface Props {
		models: Model[];
		assistant: Pick<
			Assistant,
			| "avatar"
			| "name"
			| "rag"
			| "dynamicPrompt"
			| "modelId"
			| "createdByName"
			| "exampleInputs"
			| "_id"
			| "description"
			| "userCount"
			| "tools"
		>;
	}

	let { models, assistant }: Props = $props();

	const dispatch = createEventDispatcher<{ message: string }>();

	let hasRag = $derived(
		assistant?.rag?.allowAllDomains ||
			(assistant?.rag?.allowedDomains?.length ?? 0) > 0 ||
			(assistant?.rag?.allowedLinks?.length ?? 0) > 0 ||
			assistant?.dynamicPrompt
	);

	const prefix =
		publicConfig.PUBLIC_SHARE_PREFIX || `${publicConfig.PUBLIC_ORIGIN || page.url.origin}${base}`;

	let shareUrl = $derived(`${prefix}/assistant/${assistant?._id}`);

	let isCopied = $state(false);

	const settings = useSettingsStore();
	const translations = useTranslations();
</script>

<div class="flex h-full w-full flex-col content-center items-center justify-center pb-52">
	<div
		class="relative mt-auto rounded-2xl bg-gray-100 text-gray-600 dark:border-gray-800 dark:bg-gray-800/60 dark:text-gray-300"
	>
		<div
			class="mt-3 flex min-w-[80dvw] items-center gap-4 p-4 pr-1 sm:min-w-[440px] md:p-8 xl:gap-8"
		>
			{#if assistant.avatar}
				<img
					src={`${base}/settings/assistants/${assistant._id.toString()}/avatar.jpg?hash=${
						assistant.avatar
					}`}
					alt="avatar"
					class="size-16 flex-none rounded-full object-cover max-sm:self-start md:size-32"
				/>
			{:else}
				<div
					class="flex size-12 flex-none items-center justify-center rounded-full bg-gray-300 object-cover text-xl font-bold uppercase text-gray-500 dark:bg-gray-600 max-sm:self-start sm:text-4xl md:size-32"
				>
					{assistant?.name[0]}
				</div>
			{/if}

			<div class="flex h-full flex-col gap-2 text-balance">
				<p class="-mb-1">{$translations.t("assistant")}</p>

				<p class="text-xl font-bold sm:text-2xl">{assistant.name}</p>
				{#if assistant.description}
					<p class="line-clamp-6 text-sm text-gray-500 dark:text-gray-400">
						{assistant.description}
					</p>
				{/if}

				{#if assistant?.tools?.length}
					<div
						class="flex h-5 w-fit items-center gap-1 rounded-full bg-purple-500/10 pl-1 pr-2 text-xs"
						title={$translations.t("assistant_uses_tools")}
					>
						<CarbonTools class="text-sm text-purple-600" />
						{$translations.t("has_tools")}
					</div>
				{/if}
				{#if hasRag}
					<div
						class="flex h-5 w-fit items-center gap-1 rounded-full bg-blue-500/10 pl-1 pr-2 text-xs"
						title={$translations.t("assistant_uses_websearch")}
					>
						<IconInternet classNames="text-sm text-blue-600" />
						{$translations.t("has_internet_access")}
					</div>
				{/if}

				{#if assistant.createdByName}
					<div class="pt-1 text-sm text-gray-400 dark:text-gray-500">
						{$translations.t("created_by")}
						<a class="hover:underline" href="{base}/assistants?user={assistant.createdByName}">
							{assistant.createdByName}
						</a>
						{#if assistant.userCount && assistant.userCount > 1}
							<span class="mx-1">·</span>
							<div
								class="inline-flex items-baseline gap-1 text-sm text-gray-400 dark:text-gray-500"
								title={$translations.t("number_of_users")}
							>
								<CarbonUserMultiple class="text-xxs" />{formatUserCount(assistant.userCount)}
								{$translations.t("users")}
							</div>
						{/if}
					</div>
				{/if}
			</div>
		</div>

		<div class="absolute right-3 top-3 md:right-4 md:top-4">
			<div class="flex flex-row items-center gap-1">
				<button
					class="flex h-7 items-center gap-1.5 rounded-full border bg-white px-2.5 py-1 text-gray-800 shadow-sm hover:shadow-inner dark:border-gray-700 dark:bg-gray-700 dark:text-gray-300/90 dark:hover:bg-gray-800 max-sm:px-1.5 md:text-sm"
					onclick={() => {
						if (!isCopied) {
							share(shareUrl, assistant.name);
							isCopied = true;
							setTimeout(() => {
								isCopied = false;
							}, 2000);
						}
					}}
				>
					{#if isCopied}
						<CarbonCheckmark class="text-xxs text-green-600 max-sm:text-xs" />
						<span class="text-green-600 max-sm:hidden"> {$translations.t("link_copied")} </span>
					{:else}
						<CarbonExport class="text-xxs max-sm:text-xs" />
						<span class="max-sm:hidden"> {$translations.t("share")} </span>
					{/if}
				</button>
				<a
					href="{base}/settings/assistants/{assistant._id.toString()}"
					class="flex h-7 items-center gap-1.5 rounded-full border bg-white px-2.5 py-1 text-gray-800 shadow-sm hover:shadow-inner dark:border-gray-700 dark:bg-gray-700 dark:text-gray-300/90 dark:hover:bg-gray-800 md:text-sm"
					><IconGear class="text-xxs" />{$translations.t("settings")}</a
				>
			</div>
		</div>
		<button
			onclick={() => {
				settings.instantSet({
					activeModel: models[0].name,
				});
				goto(`${base}/`);
			}}
			class="absolute -bottom-6 right-2 inline-flex items-center justify-center text-xs text-gray-600 underline hover:brightness-50 dark:text-gray-400 dark:hover:brightness-110"
		>
			<CarbonRenew class="mr-1.5 text-xxs" />
			{$translations.t("reset_to_default_model")}
		</button>
	</div>
	{#if assistant.exampleInputs}
		<div class="mx-auto mt-auto w-full gap-8 sm:-mb-8">
			<div class="md:col-span-2 md:mt-6">
				<div
					class="grid grid-cols-1 gap-3 {assistant.exampleInputs.length > 1
						? 'md:grid-cols-2'
						: ''}"
				>
					{#each assistant.exampleInputs as example}
						<button
							type="button"
							class="truncate whitespace-nowrap rounded-xl border bg-gray-50 px-3 py-2 text-left text-smd text-gray-600 hover:bg-gray-100 dark:border-gray-800 dark:bg-gray-800 dark:text-gray-300 dark:hover:bg-gray-700"
							onclick={() => dispatch("message", example)}
						>
							{example}
						</button>
					{/each}
				</div>
			</div>
		</div>
	{/if}
</div>
