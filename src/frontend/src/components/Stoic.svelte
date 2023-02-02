<script>
  import { AuthClient } from "@dfinity/auth-client";
  import { onMount } from "svelte";
  import { auth, createActor } from "../store/auth";
  import {StoicIdentity} from "ic-stoic-identity";

  /** @type {AuthClient} */
  let client;

  let whoami = $auth.actor.whoami();

  onMount(async () => {

  });

  function handleAuth(identity) {
    auth.update(() => ({
      loggedIn: true,
      actor: createActor({
        agentOptions: {
          identity: identity,
        },
      }),
    }));

    whoami = $auth.actor.whoami();
  }

  function login() {
    StoicIdentity.load().then(async identity => {
    if (identity !== false) {
      //ID is a already connected wallet!
    } else {
      //No existing connection, lets make one!
      identity = await StoicIdentity.connect();
    }
    handleAuth(identity);
    //Lets display the connected principal!
    console.log(identity.getPrincipal().toText());
    });
  }

  async function logout() {
    StoicIdentity.disconnect();
    auth.update(() => ({
      loggedIn: false,
      actor: createActor(),
    }));

    whoami = $auth.actor.whoami();
  }
</script>

<div class="container">
  {#if $auth.loggedIn}
    <div>
      <button on:click={logout}>Log out Stoic</button>
    </div>
  {:else}
    <button on:click={login}>Authenticate in with Stoic</button>
  {/if}

  <div class="principal-info">
    {#await whoami}
      Querying caller identity...
    {:then principal}
      Your principal ID is
      <code>{principal}</code>

      {#if principal.isAnonymous()}
        (anonymous)
      {/if}
    {/await}
  </div>
</div>

<style>
  .container {
    margin: 64px 0;
  }

  .principal-info {
    margin-top: 32px;
  }
</style>
