
<script lang="ts">
  import Counter from './lib/Counter.svelte'
  import Camera from './lib/Camera.svelte'
  import NFC from './lib/NFC.svelte'

  let data_card_1;
  let data_card_2;


  async function readTag1() {  
    if ("NDEFReader" in window) {
        const ndef = new NDEFReader();
        try {
        await ndef.scan();
        ndef.onreading = event => {
            const decoder = new TextDecoder();
            for (const record of event.message.records) {

              data_card_1 = decoder.decode(record.data);
              if (data_card_1){
                ndef.cancel()
                return
              }

            }
        }
        } catch(error) {

        }
        ndef.cancel()
    }
    else {

    }
  }

  async function readTag2() {  
    if ("NDEFReader" in window) {
        const ndef = new NDEFReader();
        try {
        await ndef.scan();
        ndef.onreading = event => {
            const decoder = new TextDecoder();
            for (const record of event.message.records) {

              data_card_2 = decoder.decode(record.data);
              if (data_card_2){
                ndef.cancel()
                return
              }
            }
        }
        } catch(error) {

        }
        ndef.cancel()
    } 
    else {

    }
  }


  //function handleReadTag1(data_proxy) {
  //  data_proxy = readTag(data_proxy)
  //}

</script>

<main>
  <Camera />

  {#if !data_card_1}
  <button on:click={readTag1}>First NFC card</button>
  {/if}
  
  {#if data_card_1}
    <h1>resultat : {data_card_1}</h1>
  {/if}

  {#if !data_card_1}
  <button on:click={readTag2}>Second NFC card</button>
  {/if}
  
  {#if data_card_2}
    <h1>resultat : {data_card_2}</h1>
  {/if}

  {#if data_card_1 && data_card_2}
    {#if data_card_1 != data_card_2}
      <h2>The twice cards are differnts, try again...</h2>
    {:else if data_card_1 == data_card_2}
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