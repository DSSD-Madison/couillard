<script>
  import Nav from "./components/nav/Nav.svelte"
  import TotalsBanner from "./components/TotalsBanner.svelte"
  import Footer from "./components/Footer.svelte"
  import Graph from "./components/Graph.svelte"
  import LoadingDialog from "./components/LoadingDialog.svelte"

  import { readyToAnimate } from "./stores"
  import { ANIMATE_OUT_LOADING_TIME, ANIMATE_DURATION } from "./animationTimings"


  import { onMount } from "svelte";
  import { fly } from "svelte/transition";

  let notReadyYet = true;
  onMount(() => {
    readyToAnimate.subscribe(x => {
      if (x) notReadyYet = false;
    })
  });
  
  let homeInPlace = false;
  let graphInPlace = false;

  const flyOptions = {
    y: 200,
    duration: ANIMATE_DURATION,
  };
  const flyOptionsFirst = {
    ...flyOptions,
    delay: ANIMATE_OUT_LOADING_TIME,
  }
</script>

<main>
  <LoadingDialog />
  <Nav />
  {#if !notReadyYet}
    <div in:fly={flyOptionsFirst} on:introend={() => { homeInPlace = true; }}>
      <TotalsBanner />
    </div>
  {/if}
  {#if !notReadyYet && homeInPlace}
    <div in:fly={flyOptions} on:introend={() => { graphInPlace = true; }}>
      <Graph />
      <Footer />
    </div>
  {/if}
</main>

<style>
  
</style>
