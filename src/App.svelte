
<script lang="ts">
  import Counter from './lib/Counter.svelte'
  import Camera from './lib/Camera.svelte'
  import NFC from './lib/NFC.svelte'

  let _data;

  function handleSubmit() {
    alert(`submitted ${_data}`);
  }

    async function readTag() 
    {
        if ("NDEFReader" in window) {
            const ndef = new NDEFReader();
            try {
            await ndef.scan();
            ndef.onreading = event => {
                const decoder = new TextDecoder();
                for (const record of event.message.records) {
                consoleLog("Record type:  " + record.recordType);
                consoleLog("MIME type:    " + record.mediaType);
                consoleLog("=== data ===\n" + decoder.decode(record.data));
                _data = decoder.decode(record.data);
                }
            }
            } catch(error) {
                consoleLog(error);
            }
        } 
        else {
            consoleLog("Web NFC is not supported.");
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
  {#if _data}
    <h1>resultat : {_data}</h1>
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