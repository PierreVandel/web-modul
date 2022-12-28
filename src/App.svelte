
<script lang="ts">
  import Counter from './lib/Counter.svelte'
  import Camera from './lib/Camera.svelte'
  import NFC from './lib/NFC.svelte'

  // Variables containing image link
  let image_1
  let image_2
  let image_3

  // Variables containing the first scan card data and the second scan card data
  let data_card_1;
  let data_card_2;

  // To display the result (image) after scanning cards
  let image_corresponding_to_card_1;
  let image_corresponding_to_card_2;

  // Variables to make sure all cards have been setup 
  let is_setup_1 = false;
  let is_setup_2 = false;
  let is_setup_3 = false;

  let waiting_write_tag = false;


  async function readTag(tagNumber) {  
    // Stop the camera stream before accessing the NFC reader
    theStream.getVideoTracks()[0].stop();
    if(tagNumber == 1) {
      data_card_1 = undefined;
    }

    if(tagNumber == 2) {
      data_card_2 = undefined;
    }

    if ("NDEFReader" in window) {
      
      const ndef = new NDEFReader();
      try {
        await ndef.scan();
        ndef.onreading = event => {
          const decoder = new TextDecoder();
          for (const record of event.message.records) {

            if (tagNumber == 1) {
              data_card_1 = decoder.decode(record.data);

              if (data_card_1 == "1") {
                image_corresponding_to_card_1 = image_1;
              }
              else if (data_card_1 == "2") {
                image_corresponding_to_card_1 = image_2;
              }
              else if (data_card_1 == "3") {
                image_corresponding_to_card_1 = image_3;
              }
              else {
                image_corresponding_to_card_1 = undefined;
              }
            } else if (tagNumber == 2) {
              data_card_2 = decoder.decode(record.data);

              if (data_card_1 == "1") {
                image_corresponding_to_card_2 = image_1;
              }
              else if (data_card_1 == "2") {
                image_corresponding_to_card_2 = image_2;
              }
              else if (data_card_1 == "3") {
                image_corresponding_to_card_2 = image_3;
              }
              else {
                image_corresponding_to_card_2 = undefined;
              }
            }
            ndef.cancel();
            if (data_card_1 || data_card_2) {
              ndef.cancel();
              return;
            }

          }
        }
        ndef.cancel();
      } 
      catch(error) {}
      ndef.cancel();
    }
  }

  async function writeTag(tagNumber) {
    // Stop the camera stream before accessing the NFC reader
    theStream.getVideoTracks()[0].stop();


    if ("NDEFReader" in window) {

      const ndef = new NDEFReader();
      try {
      await ndef.write(tagNumber);
      await ndef.write(tagNumber);
      } 
      catch(error) {}
      waiting_write_tag = true
      setTimeout(() => {
        
        if (tagNumber == 1) {
          is_setup_1 = true;
        } else if (tagNumber == 2) {
          is_setup_2 = true;
        } else if (tagNumber == 3) {
          is_setup_3 = true;
        }
        ndef.cancel();
        waiting_write_tag = false
      }, 10000); // Timeout for 10 secondes
    
    } else {}
  }


 //////////////////

 var imageLink = "test link";

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

function saveImage(tagNumber) {
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

  if (tagNumber == 1) {
    image_1 = imageLink;
  } else if (tagNumber == 2) {
    image_2 = imageLink;
  } else if (tagNumber == 3) {
    image_3 = imageLink;
  }

  // simulate a click on the link to trigger the download
  //link.click();
}


//let isSetup_1 = false;


</script>

<main>
  {#if !is_setup_1 || !is_setup_2 || !is_setup_3}
    {#if !waiting_write_tag}
      <p><button on:click={getStream}>Grab video</button></p>
      <p><video autoplay style="height: 720px; width: 524px;"></video></p>
      <p><button on:click={takePhoto}>Take Photo!</button></p>
      <p><img id="imageTag" width="720" height="524"></p>

      <!-- create a canvas element to draw the image to -->
      <canvas id="canvas" width="720" height="524"></canvas>

      <!-- add buttons to save the image -->

      {#if !is_setup_1}
        <button on:click={() => saveImage(1)}>Save Image 1</button>
        <button on:click={() => writeTag("1")}>Write link on NFC</button>
      {/if}


      {#if is_setup_1 && !is_setup_2}
        <button on:click={() => saveImage(2)}>Save Image 2</button>
        <button on:click={() => writeTag("2")}>Write link on NFC</button>
      {/if}


      {#if is_setup_2 && !is_setup_3}
        <button on:click={() => saveImage(3)}>Save Image 3</button>
        <button on:click={() => writeTag("3")}>Write link on NFC</button>
      {/if}
    {:else}
      <h1>You have 10 secondes to scan two NFC cards...</h1>
    {/if}
  {/if}
  <!------------------->

  

  <!--Jeu-->

  {#if is_setup_1 && is_setup_2 && is_setup_3}
    <h1>Nous pouvons jouer !!!</h1>

    <button on:click={() => readTag(1)}>First NFC card</button>
    {#if data_card_1}
      {#if image_corresponding_to_card_1}
        <img src={image_corresponding_to_card_1} alt="Card one"/>
      {:else}
        <h2>There is no image with this tag</h2>
      {/if}
    {/if}


    <button on:click={() => readTag(2)}>Second NFC card</button>
    {#if data_card_2}
      {#if image_corresponding_to_card_2}
        <img src={image_corresponding_to_card_2} alt="Card two"/>
      {:else}
        <h2>There is no image with this tag</h2>
      {/if}
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