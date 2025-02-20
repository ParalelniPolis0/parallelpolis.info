<script>
    import { addDays } from "date-fns";
    import { getContext } from "svelte";
    import { parse } from "marked";

    import { people, archive, config, projects } from "$lib/data.js";
    import { imgHashUrl } from "$lib/utils.js";

    import PeopleGrid from "$lib/components/PeopleGrid.svelte";
    import RefsBar from "$lib/components/RefsBar.svelte";
    import ArchiveItem from "$lib/components/ArchiveItem.svelte";
    import ProjectList from "$lib/components/ProjectList.svelte";

    const lang = getContext("lang");

    const { data } = $props();
    const p = $derived(data.item);
    const events = $derived(
        p.events && p.events.sort((x, y) => (x.date < y.date ? 1 : -1)),
    );
    const contributors = $derived(
        people.filter((pe) => {
            return pe.roles?.find((r) => r.project === p.id);
        }),
    );
    const instanceProjects = $derived(p.projects || []); //projects.filter((i) => i.instances.includes(p.id));
    const activeProjects = instanceProjects.filter(
        (i) => i.years && !i.years[1],
    );
    const pastProjects = instanceProjects
        .filter((i) => i.years && i.years[1])
        .sort((x, y) => (x.years[1] > y.years[1] ? -1 : 1));

    let eventView = $state("simple");

    function switchEventView() {
        if (eventView === "full") {
            eventView = "simple";
        } else {
            eventView = "full";
        }
    }
</script>

<svelte:head>
    <title>{p.name} | {config.title}</title>
</svelte:head>

<div class="flex gap-8 mt-4 flex-col-reverse sm:flex-row">
    <div class="grow">
        <div class="breadcrumb">
            <a href="/structures">Structure</a>
        </div>
        <div class="flex flex-wrap gap-2 items-center">
            <h1 class="text-4xl font-semibold">{p.name}</h1>
            <div class="font-normal text-xl opacity-50 block mt-1">
                {#if p.years}({p.years[0]} - {p.years[1]
                        ? p.years[1]
                        : ""}){/if}
            </div>
        </div>

        <RefsBar refs={p.refs} />
        {#if p.caption}
            <div class="mt-6 text-lg">{p.caption}</div>
        {/if}
    </div>
    {#if p.imgHash}
        <div class="shrink-0">
            <img
                src={imgHashUrl("structures", p.imgHash, "m")}
                alt={p.name}
                class="w-2/3 sm:w-32 md:w-48 lg:w-64 aspect-square object-cover rounded"
            />
        </div>
    {/if}
</div>

{#if contributors.length > 0}
    <div class="mt-8 mb-12">
        <h2 class="text-2xl main">
            {lang === "cs" ? "Přispěvatelé" : "Contributors"}
        </h2>
        <div class="mt-4">
            <PeopleGrid people={contributors} />
        </div>
    </div>
{/if}

{#if activeProjects.length > 0}
    <div class="mb-10 mt-4">
        <h1 class="main text-2xl">
            {lang === "cs" ? "Aktivní koncepty" : "Active concepts"}
        </h1>
        <ProjectList arr={activeProjects} />
    </div>
{/if}

{#if pastProjects.length > 0}
    <div class="mb-10 mt-4">
        <h1 class="main text-2xl">
            {lang === "cs" ? "Historické koncepty" : "Historical concepts"}
        </h1>
        <ProjectList arr={pastProjects} gray="true" />
    </div>
{/if}

{#if p.events}
    <div class="mt-8">
        <div class="flex flex-wrap items-center">
            <h2 class="grow text-2xl main">
                {lang === "cs" ? "Události" : "Events"}
            </h2>
            <div>
                <button
                    class="button pointer-cursor p-1.5 hover:bg-gray-200 hover:dark:bg-gray-800"
                    onclick={switchEventView}
                    >{#if eventView === "full"}Show simple overview{:else}Show
                        with details{/if}</button
                >
            </div>
        </div>
        <div
            class="grid grid-cols-1 {eventView === 'full'
                ? 'gap-10'
                : 'gap-8'} mt-8"
        >
            {#each events.map((e) => {
                e.archive = archive.filter((i) => i.event === e.id);
                return e;
            }) as e}
                {#if eventView === "full"}
                    <hr />
                {/if}
                <div id={e.id}>
                    <div class="flex flex-wrap">
                        <h3 class="text-3xl font-semibold grow">
                            <a href="/e/{e.id}">
                                {e.name}
                            </a>
                            {#if e.seq}<span class="opacity-50 font-normal"
                                    >(#{e.seq})</span
                                >{/if}
                        </h3>
                        <div class="text-xl opacity-50">
                            {new Date(e.date).toLocaleDateString(lang)}
                            {#if e.days && e.days > 1}
                                - {addDays(
                                    new Date(e.date),
                                    e.days - 1,
                                ).toLocaleDateString(lang)}{/if}
                        </div>
                    </div>

                    {#if eventView === "full"}
                        <div class="mt-8">
                            {#if e.speakers}
                                <PeopleGrid
                                    size="small"
                                    people={e.speakers.map((s) =>
                                        people.find((p) => p.id === s),
                                    )}
                                />
                            {/if}
                        </div>
                        {#if e.archive && e.archive.length > 0}
                            <div
                                class="mt-10 grid grid-cols-1 sm:grid-cols-3 gap-4 mb-10"
                            >
                                {#each e.archive.slice(0, 3) as item}
                                    <ArchiveItem {item} />
                                {/each}
                            </div>
                            {#if e.archive.length > 3}
                                <div class="text-right">
                                    <a href="/e/{e.id}"
                                        >More from "{e.name}" →</a
                                    >
                                </div>
                            {/if}
                        {/if}
                    {/if}
                </div>
            {/each}
        </div>
    </div>
{/if}

{#if p.description || p.description_cs}
    <div class="mt-8 mb-12">
        <h2 class="text-2xl main">{lang === "cs" ? "Popis" : "Description"}</h2>
        <div class="markdown">
            {@html parse(p.description || p.description_cs)}
        </div>
    </div>
{/if}
