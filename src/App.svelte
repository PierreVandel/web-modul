
<script lang="ts">
  import Counter from './lib/Counter.svelte'
  import Camera from './lib/Camera.svelte'
  import NFC from './lib/NFC.svelte'

  let data_card_1 = undefined;
  let data_card_2 = undefined;


  async function readTag(data) {  
    if ("NDEFReader" in window) {
        const ndef = new NDEFReader();
        try {
        await ndef.scan();
        ndef.onreading = event => {
            const decoder = new TextDecoder();
            for (const record of event.message.records) {

              data = decoder.decode(record.data);
              return data;
            }
        }
        } catch(error) {
          return undefined;
        }
    } 
    else {
      return undefined;
    }
  }

</script>

<main>
  <Camera />

  <button on:click={data_card_1 = () => readTag(data_card_1)}>First NFC card</button>
  
  {#if data_card_1}
    <h1>resultat : {data_card_1}</h1>
  {/if}

  <button on:click={data_card_2 = () => readTag(data_card_2)}>Second NFC card</button>
  
  {#if data_card_2}
    <h1>resultat : {data_card_2}</h1>
  {/if}

  {#if data_card_1 == null || data_card_2 == null}
    {#if data_card_1 != data_card_2}
      <h2>The twice cards are differnts, try again...</h2>
    {:else if data_card_1 === data_card_2}
      <h2>You find two same card</h2>
    {/if}
  {:else}
    <h2>Please scan cards</h2>
  {/if}
 
  
</main>

<style>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
</style>