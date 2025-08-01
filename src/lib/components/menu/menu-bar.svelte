<script lang="ts">
  import { controlCenter, launchpad, menuBar } from "$lib/meta.svelte";
  import dayjs from "dayjs";
  import { onMount } from "svelte";
  import TablerCircleFilled from "~icons/tabler/circle-filled";
  import TablerSun from "~icons/tabler/sun";
  import TablerMoon from "~icons/tabler/moon";
  import { activeWindow } from "../window/windows.svelte";
  import MenuBarItem from "./menu-bar-item.svelte";
  import {
    activateMenuBar,
    deactivateMenuBar,
    menuBarState,
    startDisableTimeout,
    systemActiveMenu,
    systemMenu,
    systemMenuBarItems,
  } from "./helpers.svelte";
  import ControlCenterIcon from "~icons/custom/control-center";
  import ControlCenter from "../control-center/control-center.svelte";
  import MenuBarButton from "./menu-bar-button.svelte";
  import { fade } from "svelte/transition";
  import { theme, parsedTheme } from "$lib/theme";

  let time: string = $state("");

  update();

  onMount(() => {
    setInterval(update, 1000);
  });

  function update() {
    time = dayjs().format("ddd MMM D h:mm A");
  }

  function onpointerdown(e: PointerEvent) {
    const target = e.target as HTMLElement;
    const item = target?.closest("[data-menu-bar-item]") as HTMLElement;

    if (menuBarState.visitCount > 1) {
      menuBarState.visitCount = 0;
    }

    if (item !== null) {
      const id = item.getAttribute("data-menu-bar-item");
      if (id === null) return;

      if (id !== menuBar.activeId) {
        menuBarState.visitCount = 0;
      }

      startDisableTimeout();
      activateMenuBar(id);
    } else {
      deactivateMenuBar();
    }
  }

  function onpointermove(e: PointerEvent) {
    const target = e.target as HTMLElement;
    const item = target?.closest("[data-menu-bar-item]") as HTMLElement;

    if (!menuBar.active) return;

    if (item !== null) {
      const id = item.getAttribute("data-menu-bar-item");
      if (id !== null && id !== menuBar.activeId) {
        activateMenuBar(id);
      }
    }
  }

  function onpointerup(e: PointerEvent) {
    const target = e.target as HTMLElement;
    const item = target?.closest("[data-menu-bar-item]") as HTMLElement;

    if (target.closest("[data-menu-bar-ignore]")) return;

    if (!target.closest("[data-menu]")) {
      if (menuBarState.skipPointerUp) {
        if (item === null) return;
        deactivateMenuBar();
      }

      if (menuBarState.visitCount > 0) {
        deactivateMenuBar();
      }
      menuBarState.visitCount++;
    }
  }
</script>

<svelte:document {onpointerdown} {onpointermove} {onpointerup} />

<menu
  data-menu-bar
  class={{
    "absolute flex h-8 w-full items-center bg-white/50 px-2 backdrop-blur-[50px] transition-opacity duration-500 ease-[cubic-bezier(0.22,1,0.36,1)] dark:bg-black/18": true,
    "opacity-0": launchpad.open,
    "opacity-100": !launchpad.open,
  }}
>
  <MenuBarItem type="logo" data={systemMenu}><TablerCircleFilled /></MenuBarItem>
  {#if activeWindow.menuBarActive !== null}
    <MenuBarItem type="name" data={activeWindow.menuBarActive} />
  {:else}
    <MenuBarItem type="name" data={systemActiveMenu} />
  {/if}
  {#if activeWindow.current !== null}
    {#each activeWindow.menuBarItems as data}
      <MenuBarItem {data} />
    {/each}
  {:else}
    {#each systemMenuBarItems as data}
      <MenuBarItem {data} />
    {/each}
  {/if}
  <div class="flex-1"></div>
  <MenuBarButton
    onclick={() => {
      $theme = $theme === 'light' ? 'dark' : 'light';
    }}
  >
    <span class="text-[11px]">
      {#if $parsedTheme === 'dark'}
        <TablerSun />
      {:else}
        <TablerMoon />
      {/if}
    </span>
  </MenuBarButton>
  <MenuBarButton
    controls="control-center"
    bind:active={controlCenter.open}
    onclick={() => {
      controlCenter.open = !controlCenter.open;
    }}
  >
    <span class="text-[11px]">
      <ControlCenterIcon />
    </span>
  </MenuBarButton>
  <MenuBarButton>
    {time}
  </MenuBarButton>
  {#if controlCenter.open}
    <div out:fade={{ duration: 200 }} class="absolute top-full right-0">
      <ControlCenter />
    </div>
  {/if}
</menu>
