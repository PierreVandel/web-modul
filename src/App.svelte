<script lang="ts">

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

    // Check if the NDEFReader API is available
    if ("NDEFReader" in window) {
      waiting_write_tag = true;
      const ndef = new NDEFReader();
      try {
        //Write the 2 cards with the tag
      for(let i = 0; i < 2; i++) {
        await ndef.write(tagNumber);
      }
      
      } 
      catch(error) {}

      // Update the corresponding flag for the written tag
      waiting_write_tag = false;
      if (tagNumber == 1) {
        is_setup_1 = true;
      } else if (tagNumber == 2) {
        is_setup_2 = true;
      } else if (tagNumber == 3) {
        is_setup_3 = true;
      }
      // Cancel the NFC reading process
      ndef.cancel();
      
    // If the NDEFReader API is not available, do nothing
    } else {}

  }

// Function to get the user's media (in this case, video)
function getUserMedia(options, successCallback, failureCallback) {
  // Check for various vendor prefixes of the getUserMedia function
  var api = navigator.getUserMedia || navigator.webkitGetUserMedia ||
    navigator.mozGetUserMedia || navigator.msGetUserMedia;
  // If the getUserMedia function is available, bind it to the navigator object and call it with the specified arguments
  if (api) {
    return api.bind(navigator)(options, successCallback, failureCallback);
  }
}

var theStream;
// Function to start getting the video stream
function getStream() {
  // Check if the getUserMedia function is not available
  if (!navigator.getUserMedia && !navigator.webkitGetUserMedia &&
    !navigator.mozGetUserMedia && !navigator.msGetUserMedia) {
    alert('User Media API not supported.');
    return;
  }
  
  // Set the video constraint to true
  var constraints = {
    video: true
  };

  // Call the getUserMedia function with the video constraint and specified callback functions
  getUserMedia(constraints, function (stream) {
    // Get the video element and set its source object to the stream
    var mediaControl = document.querySelector('video');
    if ('srcObject' in mediaControl) {
        mediaControl.srcObject = stream;
    // For older versions of Firefox, set the mozSrcObject property of the video element
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

  let imageLink

  imageLink = link.href = dataURL;

  if (tagNumber == 1) {
    image_1 = imageLink;
  } else if (tagNumber == 2) {
    image_2 = imageLink;
  } else if (tagNumber == 3) {
    image_3 = imageLink;
  }
}

</script>

<main>
  {#if !is_setup_1 || !is_setup_2 || !is_setup_3}
    {#if !waiting_write_tag}
      <button on:click={getStream}>Grab video</button>
      <video autoplay ></video>
      <button on:click={takePhoto}>Take Photo!</button>
      <img id="imageTag">

      <!-- create a canvas element to draw the image to -->
      <canvas id="canvas"></canvas>

      <!-- add buttons to save the image -->
      

      {#if !is_setup_1}
        <button on:click={() => saveImage(1)}>Save Image 1</button>
        <button on:click={() => writeTag("1")}>Write tag to link the photo</button>
      {/if}


      {#if is_setup_1 && !is_setup_2}
        <button on:click={() => saveImage(2)}>Save Image 2</button>
        <button on:click={() => writeTag("2")}>Write tag to link the photo</button>
      {/if}


      {#if is_setup_2 && !is_setup_3}
        <button on:click={() => saveImage(3)}>Save Image 3</button>
        <button on:click={() => writeTag("3")}>Write tag to link the photo</button>
      {/if}
    {:else}
      <h1>Please scan two NFC cards...</h1>
    {/if}
  {/if}
  

  <!--Jeu-->

  {#if is_setup_1 && is_setup_2 && is_setup_3}
    <h1>We can play !</h1>

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
        <h2>The twice cards are differents, try again...</h2>
      {:else if data_card_1 == data_card_2}
        <h2>You find two same card</h2>
      {/if}
    {:else}
      <h2>Please scan cards</h2>
    {/if}
  {/if}
  
</main>

<style>
  button {
    width:100%;
  }

  video {
    width:100%;
  }

  img {
    width:100%;
  }

  canvas {
    width:100%;
  }

  .read-the-docs {
    color: rgb(150, 150, 150);
  }
</style>