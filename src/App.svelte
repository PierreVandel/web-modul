
<script lang="ts">
  import Counter from './lib/Counter.svelte'
  import Camera from './lib/Camera.svelte'
  import NFC from './lib/NFC.svelte'

  let data_card_1;
  let data_card_2;


  async function readTag(tagNumber) {  
  if ("NDEFReader" in window) {
      const ndef = new NDEFReader();
      try {
      await ndef.scan();
      ndef.onreading = event => {
          const decoder = new TextDecoder();
          for (const record of event.message.records) {

            if (tagNumber === 1) {
              data_card_1 = decoder.decode(record.data);
            } else if (tagNumber === 2) {
              data_card_2 = decoder.decode(record.data);
            }

            if (data_card_1 || data_card_2) {
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

async function writeTag(link) {
    if (link && "NDEFReader" in window) {
        const ndef = new NDEFReader();
        try {
            await ndef.write(link);
            consoleLog("NDEF message written!");
        } catch(error) {
            consoleLog(error);
        }
    } else {
        consoleLog("Web NFC is not supported or link is not defined.");
    }
}


 //////////////////

 var imageLink;

function getUserMedia(options, successCallback, failureCallback) {
    var api = navigator.getUserMedia || navigator.webkitGetUserMedia ||
        navigator.mozGetUserMedia || navigator.msGetUserMedia;
    if (api) {
        return api.bind(navigator)(options, successCallback, failureCallback);
    }
}

var theStream;

function getStream() {
    if (!navigator.getUserMedia && !navigator.webkitGetUserMedia &&
        !navigator.mozGetUserMedia && !navigator.msGetUserMedia) {
        alert('User Media API not supported.');
        return;
    }
    
    var constraints = {
        video: true
    };

    getUserMedia(constraints, function (stream) {
        var mediaControl = document.querySelector('video');
        if ('srcObject' in mediaControl) {
            mediaControl.srcObject = stream;
        } else if (navigator.mozGetUserMedia) {
            mediaControl.mozSrcObject = stream;
        } else {
            mediaControl.src = (window.URL || window.webkitURL).createObjectURL(stream);
        }
        theStream = stream;
    }, function (err) {
        alert('Error: ' + err);
    });
}

function takePhoto() {
    if (!('ImageCapture' in window)) {
        alert('ImageCapture is not available');
        return;
    }
    
    if (!theStream) {
        alert('Grab the video stream first!');
        return;
    }
    
    var theImageCapturer = new ImageCapture(theStream.getVideoTracks()[0]);

    theImageCapturer.takePhoto()
        .then(blob => {
            var theImageTag = document.getElementById("imageTag");
            theImageTag.src = URL.createObjectURL(blob);
        })
        .catch(err => alert('Error: ' + err));
}
function saveImage() {
// get the canvas element and its context
var canvas = document.getElementById('canvas');
var context = canvas.getContext('2d');

// draw the image to the canvas
var img = document.getElementById('imageTag');
context.drawImage(img, 0, 0);

// get the base64-encoded data URL of the image
var dataURL = canvas.toDataURL('image/png');

// create a temporary link to the image
var link = document.createElement('a');
link.download = 'image.png';
imageLink = link.href = dataURL;

// simulate a click on the link to trigger the download
link.click();
}

</script>

<main>
  
  <p><button on:click={getStream}>Grab video</button></p>
  <p><video autoplay style="height: 180px; width: 240px;"></video></p>
  <p><button on:click={takePhoto}>Take Photo!</button></p>
  <p><img id="imageTag" width="240" height="180"></p>

  <!-- create a canvas element to draw the image to -->
  <canvas id="canvas" width="720" height="524"></canvas>

  <!-- add a button to save the image -->
  <button on:click={saveImage}>Save Image</button>

  <!------------------->
  <!--Enregistrement de l'url sur les cartes-->
  <button on:click={() => writeTag(imageLink)}>Write link on NFC</button>

  <!--Jeu-->
  <button on:click={() => readTag(1)}>First NFC card</button>
  
  {#if data_card_1}
    <h1>resultat : {data_card_1}</h1>
  {/if}

  <button on:click={() => readTag(2)}>Second NFC card</button>
  
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