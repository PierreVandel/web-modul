<script>

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



<p><button on:click={getStream}>Grab video</button></p>
<p><video autoplay style="height: 180px; width: 240px;"></video></p>
<p><button on:click={takePhoto}>Take Photo!</button></p>
<p><img id="imageTag" width="240" height="180"></p>

<!-- create a canvas element to draw the image to -->
<canvas id="canvas" width="720" height="524"></canvas>

<!-- add a button to save the image -->
<button on:click={saveImage}>Save Image</button>

{#if imageLink}
    <h3>{imageLink}</h3>
{/if}
