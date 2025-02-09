<script lang="ts">
  import { goto } from "$app/navigation";
  import Icon from "@/lib/Icon.svelte";
  import Poster from "@/lib/poster/Poster.svelte";
  import PosterList from "@/lib/poster/PosterList.svelte";
  import { activeFilters, activeSort } from "@/store";
  import type { Watched } from "@/types";

  export let list: Watched[];
  export let isPublicList: boolean = false;

  $: sort = $activeSort;
  $: filters = $activeFilters;
  $: watched = list
    .sort((a, b) => {
      if (sort[0] === "DATEADDED" && sort[1] === "UP") {
        return Date.parse(a.createdAt) - Date.parse(b.createdAt);
      } else if (sort[0] === "ALPHA") {
        if (sort[1] === "UP") return a.content.title.localeCompare(b.content.title);
        else if (sort[1] === "DOWN") return b.content.title.localeCompare(a.content.title);
      } else if (sort[0] === "LASTCHANGED") {
        if (sort[1] === "UP") return Date.parse(a.updatedAt) - Date.parse(b.updatedAt);
        else if (sort[1] === "DOWN") return Date.parse(b.updatedAt) - Date.parse(a.updatedAt);
      } else if (sort[0] === "RATING") {
        if (sort[1] === "UP") return (a.rating ?? 0) - (b.rating ?? 0);
        else if (sort[1] === "DOWN") return (b.rating ?? 0) - (a.rating ?? 0);
      }
      // default DATEADDED DOWN
      return Date.parse(b.createdAt) - Date.parse(a.createdAt);
    })
    .filter((w) => {
      if (filters.status.length <= 0 && filters.type.length <= 0) return w;
      if (filters.status.length > 0 && filters.type.length > 0) {
        return (
          filters.status.includes(w.status?.toLowerCase()) && filters.type.includes(w.content.type)
        );
      }
      if (filters.type.length > 0) {
        return filters.type.includes(w.content.type);
      }
      if (filters.status.length > 0) {
        return filters.status.includes(w.status?.toLowerCase());
      }
    });
</script>

<PosterList>
  {#if watched?.length > 0}
    {#each watched as w (w.id)}
      <Poster
        id={w.id}
        media={{
          id: w.content.tmdbId,
          poster_path: w.content.poster_path,
          title: w.content.title,
          overview: w.content.overview,
          media_type: w.content.type,
          release_date: w.content.release_date,
          first_air_date: w.content.first_air_date
        }}
        rating={w.rating}
        status={w.status}
        disableInteraction={isPublicList}
      />
    {/each}
  {:else}
    <div class="empty-list">
      <Icon i="reel" wh={80} />
      {#if isPublicList}
        <h2 class="norm">This watched list is empty!</h2>
        <h4 class="norm">Come back later to see if they have added anything.</h4>
      {:else}
        <h2 class="norm">Your watched list is empty!</h2>
        <h4 class="norm">Try searching for something you would like to add.</h4>
        <button on:click={() => goto("/import")}>Import</button>
      {/if}
    </div>
  {/if}
</PosterList>

<style lang="scss">
  .empty-list {
    display: flex;
    flex-flow: column;
    gap: 5px;
    align-items: center;

    h2 {
      margin-top: 10px;
    }

    h4 {
      font-weight: normal;
    }

    button {
      width: max-content;
      padding-left: 20px;
      padding-right: 20px;
      margin-top: 15px;
    }
  }
</style>
