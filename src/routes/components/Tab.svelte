<script>
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();
  import Fa from "svelte-fa";

  export let navList, activeItem;
  let menu, menuBorder, thisActive;
  let menuItems = [];

  function offsetMenuBorder() {
    if (!menu || !menuItems || !menuBorder) return;

    const offsetActiveItem = thisActive.getBoundingClientRect();
    const left =
      Math.floor(
        offsetActiveItem.left -
          menu.offsetLeft -
          (menuBorder.offsetWidth - offsetActiveItem.width) / 2
      ) + "px";
    menuBorder.style.transform = `translate3d(${left}, 0, 0)`;
  }

  // Execute offsetMenuBorder after update to ensure elements are available
  import { afterUpdate } from "svelte";
  afterUpdate(() => {
    offsetMenuBorder();
  });
</script>

<svelte:window on:resize={offsetMenuBorder} />

<div id="my-body">
  <header>
    <div id="menu-box">
      <menu class="menu" bind:this={menu}>
        {#each navList as item}
          {#if item == activeItem}
            <button
              class="menu__item active"
              on:click={(e) => dispatch("tabChange", item)}
              bind:this={thisActive}
            >
              <Fa icon={item.icon} size="18" />
              {item.name}
            </button>
          {:else}
            <button
              class="menu__item"
              on:click={(e) => dispatch("tabChange", item)}
            >
              <Fa icon={item.icon} />
              {item.name}
            </button>
          {/if}
        {/each}

        <div class="menu__border" bind:this={menuBorder} />
      </menu>

      <div class="svg-container">
        <svg viewBox="0 0 202.9 45.5">
          <clipPath
            id="menu"
            clipPathUnits="objectBoundingBox"
            transform="scale(0.0049285362247413 0.021978021978022)"
          >
            <path
              d="M6.7,45.5c5.7,0.1,14.1-0.4,23.3-4c5.7-2.3,9.9-5,18.1-10.5c10.7-7.1,11.8-9.2,20.6-14.3c5-2.9,9.2-5.2,15.2-7
                  c7.1-2.1,13.3-2.3,17.6-2.1c4.2-0.2,10.5,0.1,17.6,2.1c6.1,1.8,10.2,4.1,15.2,7c8.8,5,9.9,7.1,20.6,14.3c8.3,5.5,12.4,8.2,18.1,10.5
                  c9.2,3.6,17.6,4.2,23.3,4H6.7z"
            />
          </clipPath>
        </svg>
      </div>
    </div>
  </header>
  <hr />
  <div class="wrapper">
    <slot />
  </div>

  <footer>
    <p>
      Developed by: <a
        href="https://www.linkedin.com/in/marcelohas/"
        target="_blank"
        rel="noopener noreferrer">Marcelo Has</a
      >
    </p>
  </footer>
</div>

<style>
  #my-body {
    background-color: var(--off-white);
    padding: 40px;
    height: 100%;
    min-width: 580px;
    min-height: 540px;
  }
  footer {
    font-size: 12px;
    color: #404040;
    text-align: center;
    margin-top: 50px;
    display: flex;
    align-items: flex-end;
    justify-content: center;
    flex: 1;
  }
  hr {
    margin: 0px;
    margin: 0px;
    background: var(--light-purple);
  }

  .wrapper {
    display: flex;
    flex-direction: column;
    padding: 40px 30px !important;
    gap: 24px;
    background: #ffffff;
    border-radius: 0 0 16px 16px;
    width: 100%;
    margin: auto;
    box-shadow: 0px 4px 12px rgba(0, 0, 0, 0.06);
    height: calc(100vh - 249px);
    min-height: inherit;
  }

  @keyframes slideaway {
    from {
      transform: translateY(40px);
      opacity: 0;
    }
  }

  .active {
    animation: slideaway;
  }

  header {
    width: 100%;
    --duration: 0.7s;
    box-shadow: 0px 4px 12px rgba(0, 0, 0, 0.06);
  }

  .menu {
    display: flex;
    position: relative;
    padding: 10px;
    background-color: var(--white);
    border-radius: 16px 16px 0 0;
    font-size: 14px;
    color: var(--medium-black);
  }

  .menu__item {
    all: unset;
    flex-grow: 1;
    z-index: 100;
    display: flex;
    flex-direction: column;
    gap: 10px;
    cursor: pointer;
    position: relative;
    border-radius: 50%;
    align-items: center;
    will-change: transform;
    justify-content: center;
    padding: 0.55em 0 0.85em;
    transition: transform var(--timeOut, var(--duration));
  }

  .menu__item::before {
    content: "";
    margin-top: -40px;
    z-index: -1;
    width: 50px;
    height: 50px;
    border-radius: 50%;
    position: absolute;
    transform: scale(0);
    transition: background-color var(--duration), transform var(--duration);
    box-shadow: 0px 4px 12px rgba(0, 0, 0, 0.2);
  }

  .menu__item.active {
    font-weight: 600;
    transform: translate3d(0, -0.8em, 0);
    gap: 20px;
    color: black;
  }

  .menu__item.active::before {
    transform: scale(0.85);
    background-color: var(--medium-purple);
  }

  .menu__border {
    left: 0;
    bottom: 99%;
    width: 10.9em;
    height: 30px;
    position: absolute;
    clip-path: url(#menu);
    will-change: transform;
    background-color: var(--white);
    transition: transform var(--timeOut, var(--duration));
  }

  .svg-container {
    width: 0;
    height: 0;
  }
</style>
