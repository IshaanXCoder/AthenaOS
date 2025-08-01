<script lang="ts">
  import { controlCenter, search } from "$lib/meta.svelte";
  import { onMount } from "svelte";
  import HeroiconsArrowsPointingOut from "~icons/heroicons/arrows-pointing-out";
  import HeroiconsArrowsPointingIn from "~icons/heroicons/arrows-pointing-in";
  import HeroiconsMagnifyingGlass from "~icons/heroicons/magnifying-glass";
  import HeroiconsMoon16Solid from "~icons/heroicons/moon-16-solid";
  import HeroiconsSignal16Solid from "~icons/heroicons/signal-16-solid";
  import HeroiconsSpeakerWave16Solid from "~icons/heroicons/speaker-wave-16-solid";
  import HeroiconsSun16Solid from "~icons/heroicons/sun-16-solid";
  import HeroiconsWifi16Solid from "~icons/heroicons/wifi-16-solid";
  import TablerBluetooth from "~icons/tabler/bluetooth";
  import ControlCenterButton from "./control-center-button.svelte";
  import ControlCenterToggle from "./control-center-toggle.svelte";

  let isFullscreen = false;

  function updateFullscreenState() {
    isFullscreen = !!document.fullscreenElement;
  }

  onMount(() => {
    updateFullscreenState();
    document.addEventListener('fullscreenchange', updateFullscreenState);
    
    return () => {
      document.removeEventListener('fullscreenchange', updateFullscreenState);
    };
  });

  function toggleFullscreen() {
    controlCenter.open = false;
    
    try {
      if (document.fullscreenElement) {
        document.exitFullscreen()
          .then(() => {
            isFullscreen = false;
          })
          .catch(err => {
            console.error('Error exiting fullscreen:', err);
          });
      } else {
        document.documentElement.requestFullscreen()
          .then(() => {
            isFullscreen = true;
          })
          .catch(err => {
            console.error('Error entering fullscreen:', err);
          });
      }
    } catch (err) {
      console.error('Fullscreen API error:', err);
    }
  }

  function onpointerdown(e: PointerEvent) {
    const target = e.target as HTMLElement;
    if (
      !target.closest("[data-control-center]") &&
      !target.closest(`[data-control-center-controls="control-center"]`) &&
      controlCenter.open
    ) {
      controlCenter.open = false;
    }
  }
</script>

<svelte:document {onpointerdown} />

<div
  data-control-center
  class="bg-glass fixed right-0 m-1.5 grid w-80 cursor-default grid-cols-4 gap-2.5 rounded-[17.5px] p-2.5 shadow-lg"
>
  <div class="bg-control col-span-2 col-start-1 flex h-33 flex-col justify-center gap-2 pl-3">
    <ControlCenterToggle primary="Wi-Fi" secondary="localhost" disabled>
      <HeroiconsWifi16Solid />
    </ControlCenterToggle>
    <ControlCenterToggle primary="Bluetooth" secondary="On">
      <TablerBluetooth />
    </ControlCenterToggle>
    <ControlCenterToggle primary="Sharing" secondary="Everyone">
      <HeroiconsSignal16Solid />
    </ControlCenterToggle>
  </div>
  <div class="col-span-2 col-start-3 grid h-33 grid-rows-2 gap-2.5">
    <div class="bg-control col-span-2 flex flex-col justify-center pl-3">
      <ControlCenterToggle primary="Focus" value={false}>
        <HeroiconsMoon16Solid />
      </ControlCenterToggle>
    </div>
    <div class="col-span-2 grid grid-cols-2 gap-2.5">
      <ControlCenterButton
        label="Search"
        onclick={() => {
          controlCenter.open = false;
          search.current?.show();
        }}
      >
        <HeroiconsMagnifyingGlass />
      </ControlCenterButton>
      <ControlCenterButton
        label={isFullscreen ? "Exit Fullscreen" : "Fullscreen"}
        onclick={toggleFullscreen}
      >
        {#if isFullscreen}
          <HeroiconsArrowsPointingIn />
        {:else}
          <HeroiconsArrowsPointingOut />
        {/if}
      </ControlCenterButton>
    </div>
  </div>
  <div class="bg-control col-span-4 flex h-16.5 flex-col justify-center gap-2 px-3">
    <p class="-mt-1 text-[12px] font-semibold">Display</p>
    <div class="relative flex">
      <input
        type="range"
        min="0"
        max="100"
        value="60"
        class="range [--range-progress:white] [--range-thumb:white]"
      />
      <span
        class="pointer-events-none absolute top-1/2 left-0.75 -translate-y-1/2 text-[10px] text-black/80"
      >
        <HeroiconsSun16Solid />
      </span>
    </div>
  </div>
  <div class="bg-control col-span-4 flex h-16.5 flex-col justify-center gap-2 px-3">
    <p class="-mt-1 text-[12px] font-semibold">Sound</p>
    <div class="relative flex">
      <input
        type="range"
        min="0"
        max="100"
        value="40"
        class="range [--range-progress:white] [--range-thumb:white]"
      />
      <span
        class="pointer-events-none absolute top-1/2 left-0.75 -translate-y-1/2 text-[10px] text-black/80"
      >
        <HeroiconsSpeakerWave16Solid />
      </span>
    </div>
  </div>
</div>
