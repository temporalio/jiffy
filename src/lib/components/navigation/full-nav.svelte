<script lang="ts">
  import Icon from '$lib/components/icon/index.svelte';
  import type { User } from '$lib/global';

  import { navOpen } from '$lib/stores/nav-open';
  import { lastUsedNamespace } from '$lib/stores/namespaces';

  import NavContainer from './_nav-container.svelte';
  import NavRow from './_nav-row.svelte';
  import Logout from '$lib/components/logout/index.svelte';
  import NamespaceList from '$lib/components/namespace-list/index.svelte';
  import Drawer from '$lib/components/navigation/_drawer.svelte';
  import Tooltip from '$lib/components/tooltip/index.svelte';
  import IsCloudGuard from '$lib/components/guard/is-cloud-guard.svelte';

  import type { routes } from '$lib/routes';
  import type { ExtraIcon, NamespaceItem } from '$lib/global';
  import { afterNavigate } from '$app/navigation';

  export let isCloud = false;
  export let activeNamespace: string | null | undefined;
  export let namespaceList: null | Promise<NamespaceItem[]> = null;
  export let linkList: Partial<Record<keyof typeof routes, string>>;
  export let user: null | Promise<User> = null;
  export let logout: () => void;
  export let extras: ExtraIcon[] | null = null;

  let showProfilePic = true;
  let namespaceSelectorOpen: boolean | null = null;

  function fixImage() {
    showProfilePic = false;
  }

  function toggleNamespaceSelector() {
    namespaceSelectorOpen = !namespaceSelectorOpen;
  }

  afterNavigate(() => {
    if (namespaceSelectorOpen) {
      namespaceSelectorOpen = false;
    }
  });
</script>

<NavContainer {isCloud} {linkList}>
  <svelte:fragment slot="top">
    <NavRow {isCloud} wrap data-cy="namespace-select-button">
      <div
        class="relative flex cursor-pointer items-center"
        on:click={toggleNamespaceSelector}
      >
        <Tooltip right hide={$navOpen} text={activeNamespace ?? 'Namespaces'}>
          <div class="nav-icon">
            <Icon name="namespaceSelect" scale={1.6} />
          </div>
        </Tooltip>
        <div class="nav-title namespace">
          {activeNamespace ?? 'Namespaces'}
        </div>
      </div>
    </NavRow>
    <NavRow link={linkList.workflows} {isCloud} data-cy="worfklows-button">
      <Tooltip right hide={$navOpen} text="Workflows">
        <div class="nav-icon">
          <Icon name="workflow" scale={1.5} />
        </div>
      </Tooltip>
      <div class="nav-title">Workflows</div>
    </NavRow>
    <IsCloudGuard {isCloud}>
      <NavRow link={linkList.archive} {isCloud} data-cy="archive-button">
        <Tooltip right hide={$navOpen} text="Archive">
          <div class="nav-icon">
            <Icon name="archive" scale={1.2} />
          </div>
        </Tooltip>
        <div class="nav-title">Archive</div>
      </NavRow>
    </IsCloudGuard>
  </svelte:fragment>
  <svelte:fragment slot="bottom">
    {#if extras}
      {#each extras as extra}
        <NavRow {isCloud} noFilter>
          <div class="nav-icon">
            <svelte:component this={extra.component} />
          </div>
          <div class="nav-title">
            {extra.name}
          </div>
        </NavRow>
      {/each}
    {/if}
    <IsCloudGuard {isCloud}>
      <NavRow link={linkList.settings} {isCloud} data-cy="settings-button">
        <Tooltip right hide={$navOpen} text="Settings">
          <div class="nav-icon">
            <Icon name="settings" scale={1.4} />
          </div>
        </Tooltip>
        <div class="nav-title">Settings</div>
      </NavRow>
    </IsCloudGuard>
    <NavRow link={linkList.feedback} {isCloud} externalLink>
      <Tooltip right hide={$navOpen} text="Feedback">
        <div class="nav-icon">
          <Icon name="feedback" scale={1.4} />
        </div>
      </Tooltip>
      <div class="nav-title">Feedback</div>
    </NavRow>
    {#await user}
      <NavRow {isCloud}>
        <div class="motion-safe:animate-pulse" style="margin-left:1rem">
          <div class="rounded-full bg-blueGray-200 h-full aspect-square" />
        </div>
        <div class="nav-title">
          <div class="h-2 bg-blueGray-50 rounded mt-1" />
        </div>
      </NavRow>
    {:then user}
      {#if user?.email}
        <NavRow {isCloud}>
          <Tooltip right hide={$navOpen} text="Logout">
            <div class="nav-icon" on:click={logout}>
              <Icon name="logout" scale={1.4} />
            </div>
          </Tooltip>
          <div class="nav-title"><Logout {logout} /></div>
        </NavRow>
        <div class="profile-row">
          <div>
            {#if user?.picture}
              <img
                src={user?.picture}
                alt={user?.profile}
                class="rounded-md p-1 w-8 h-8"
                on:error={fixImage}
                class:hidden={!showProfilePic}
              />
              <div
                class="rounded-full p-0.5 bg-blue-200 h-full aspect-square"
                class:hidden={showProfilePic}
              >
                {#if user?.name}
                  <div class="text-black text-center ">
                    {user?.name.trim().charAt(0)}
                  </div>
                {/if}
              </div>
            {/if}
          </div>
          <div class="nav-title truncate">
            {#if user?.name}
              {user?.name}
            {/if}
          </div>
        </div>
      {/if}
    {/await}
  </svelte:fragment>
  <svelte:fragment slot="drawer">
    <Drawer
      flyin={namespaceSelectorOpen === true}
      flyout={namespaceSelectorOpen === false}
      onClose={() => {
        if (namespaceSelectorOpen === true) namespaceSelectorOpen = false;
      }}
    >
      {#if namespaceSelectorOpen}
        <NamespaceList
          {namespaceList}
          {activeNamespace}
          lastUsedNamespace={$lastUsedNamespace}
        />
      {/if}
    </Drawer>
  </svelte:fragment>
</NavContainer>

<style lang="postcss">
  .nav-icon {
    @apply ml-2 mr-2 mt-0 h-6 cursor-pointer;
  }
  .nav-title {
    width: 100px;
    overflow: hidden;
    transition: width 0.25s linear;
    -webkit-transition: width 0.25s linear;
  }
  .profile-row {
    @apply ml-1 flex flex-row items-center rounded-lg py-1 font-secondary text-sm font-medium;
  }
</style>
