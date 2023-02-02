<script>
import { onMount } from "svelte";
import { auth, createActor, createPlugActor } from "../store/auth";
import { idlFactory } from "../../../declarations/backend/backend.did.js";

let agent = undefined;

let whoami = $auth.actor.whoami();

onMount(async () => {
    console.log("plug start");
  });

async function getActor(canisterId, idl) {
    const actor = await window.ic.plug.createActor({
      canisterId,
      interfaceFactory: idl,
    });  
    return actor
}

async function login(whitelist) {
    if (window.ic === undefined) {
    window.open('https://plugwallet.ooo/', '_blank')?.focus();
    
    return
    }

    const connected = await window.ic.plug.isConnected();
    // const host = getHost();
    // const ids = getCanisterIds();
    whitelist = [process.env.BACKEND_CANISTER_ID];

    if (!connected) {
      const result = await window.ic.plug.requestConnect();

      if (!result) {
          console.log('Could not login to plug');
          return;
      }
    }
    
    // await window.ic.plug.createAgent({ whitelist, host:"https://boundary.ic0.app/" })
    // agent = window.ic.plug.agent;
    // // agent.fetchRootKey();
    // const principal = await agent.getPrincipal();
    
    let actor = await createPlugActor();
    console.log(actor);
    auth.update(() => ({
      loggedIn: true,
      actor: actor,
    }));
    whoami = $auth.actor.whoami();
    // auth.setAgent(agent);
    // auth.setPrincipal(principal);
}

async function logout() {
  auth.update(() => ({
      loggedIn: false,
      actor: createActor(),
    }));

  whoami = $auth.actor.whoami();
    // auth.setAgent(undefined);
    // auth.setPrincipal(undefined);
}

async function requestTransfer(data){
    if (window.ic === undefined) return;

    return await window.ic.plug.requestTransfer(data);
}

async function getBalance() {
    const result = await window.ic.plug.requestBalance();

    return result;
}

    // return {
    //     name: 'plug',
    //     logIn,
    //     logOut,
    //     getActor,
    //     requestTransfer,
    //     getBalance
    //   };
</script>

<div class="container">
  {#if $auth.loggedIn}
    <div>
      <button on:click={logout}>Log out Plug</button>
    </div>
  {:else}
    <button on:click={login}>Authenticate in with Plug</button>
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
