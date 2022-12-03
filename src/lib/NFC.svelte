<script>
    import {createEventDispatcher} from 'svelte';

    export let data;

    const dispatch = createEventDispatcher();

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
                dispatch('data', decoder.decode(record.data))
                
                }
            }
            } catch(error) {
                consoleLog(error);
            }
        } else {
            consoleLog("Web NFC is not supported.");
            }
    }

    async function writeTag() {
    if ("NDEFReader" in window) {
        const ndef = new NDEFReader();
        try {
        await ndef.write("Hello Pierre");
        consoleLog("NDEF message written!");
        } catch(error) {
            consoleLog(error);
        }
    } else {
        consoleLog("Web NFC is not supported.");
        }
    }

    function consoleLog(data) {
    var logElement = document.getElementById('log');
    logElement.innerHTML += data + '\n';
    }
</script>

<p>
    <button on:click={readTag}>Test NFC Read</button>
    <button on:click={writeTag}>Test NFC Write</button>
  </p>
  <pre id="log"></pre>
  <p><small>Based on the code snippets from <a href="https://w3c.github.io/web-nfc/#examples">specification draft</a>.</small></p>