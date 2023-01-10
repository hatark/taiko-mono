<script lang="ts">
  import type { ethers } from "ethers";
  import { displayStatusValue } from "../utils/displayStatusValue";
  import { onDestroy, onMount } from "svelte";
  import Loader from "../components/Loader.svelte";
  import type Status from "../domain/status";

  export let provider: ethers.providers.JsonRpcProvider;
  export let contractAddress: string;

  export let statusFunc: (
    provider: ethers.providers.JsonRpcProvider,
    contractAddress: string
  ) => Promise<Status>;

  export let watchStatusFunc: (
    provider: ethers.providers.JsonRpcProvider,
    contractAddress: string,
    onEvent: (value: Status) => void
  ) => void;

  export let colorFunc: (value: Status) => string;

  export let onClick: (value: Status) => void = null;

  export let header: string;

  export let intervalInMs: number = 0;

  let interval: NodeJS.Timer;

  let statusValue: Status;

  let loading: boolean = false;

  onMount(async () => {
    try {
      loading = true;
      statusValue = await statusFunc(provider, contractAddress);
    } catch (e) {
      console.error(e);
    } finally {
      loading = false;
    }

    if (watchStatusFunc) {
      watchStatusFunc(provider, contractAddress, (value: Status) => {
        statusValue = value;
      });
    }

    if (intervalInMs !== 0) {
      interval = setInterval(async () => {
        try {
          loading = true;
          statusValue = await statusFunc(provider, contractAddress);
        } catch (e) {
          console.error(e);
        } finally {
          loading = false;
        }
      }, intervalInMs);
    }
  });

  onDestroy(() => {
    if (interval) clearInterval(interval);
  });
</script>

<div class="rounded-3xl border-2 border-zinc-800 border-solid p-4">
  <h2 class="font-bold">{header}</h2>
  {#if loading}
    <Loader />
  {:else if statusValue || typeof statusValue === "number"}
    <span
      class={onClick ? "cursor-pointer" : ""}
      on:click={() => onClick(statusValue)}
    >
      <span class={colorFunc(statusValue)}>
        {displayStatusValue(statusValue)}
      </span>
    </span>
  {/if}
</div>