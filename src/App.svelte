
<script lang="ts">
  import Counter from './lib/Counter.svelte'
  import Camera from './lib/Camera.svelte'
  import NFC from './lib/NFC.svelte'

  let _data;

  async function readTag() {
    if ("NDEFReader" in window) {
        const ndef = new NDEFReader();
        try {
        await ndef.scan();
        ndef.onreading = event => {
            const decoder = new TextDecoder();
            for (const record of event.message.records) {

            _data = decoder.decode(record.data);
            }
        }
        } catch(error) {

        }
    } 
    else {

    }
  }

</script>

<main>
  <Camera />

  <!--
    <NFC bind:data={_data} on:submit={handleSubmit}/>
  
  <script>_data = _data</script>
  -->
  <button on:click={readTag}>Test NFC Read</button>
  <h1>resultat : {_data}</h1>
  {#if _data}
    <script>handleSubmit()</script>
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