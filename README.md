# Test

<canvas id="canvas"></canvas>
<p>
  <button type="button" id="clear-button">Clear Canvas</button>
  <button type="button" id="imgulr-button">Copy img URL</button>
  <button type="button" id="predict-button">Predict</button>
</p>
<p>Predicted Number: <span id="predicted-number"></span></p>
<p>Confidence: <span id="confidence"></span></p>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs@2.0.0/dist/tf.min.js"></script>
<script>
  window.addEventListener("load", () => {
    //HTML elements
    const canvas = document.querySelector("#canvas");
    const context = canvas.getContext("2d");
    const clearButton = document.querySelector("#clear-button");
    const urlButton = document.querySelector("#imgulr-button");
    const predictButton = document.querySelector("#predict-button");
    const predDisplay = document.querySelector("#predicted-number");
    const confDisplay = document.querySelector("#confidence");
    
    //Loading model
    let model;
    (async function () {
      console.log("Loading model...");
      model = await tf.loadLayersModel("model/model.json");
      console.log("Model loaded...");
    })();
  
    canvas.height = 300;
    canvas.width = 300;

    context.fillStyle = "black";
    context.fillRect(0, 0, canvas.width, canvas.height);

    //variables
    let painting = false;

    function startPosition(e){
      painting = true;
      draw(e);
    }

    function finishPosition(){
      painting = false;
      context.beginPath();
    }
    
    function getMousePos(canvas, e) {
      var rect = canvas.getBoundingClientRect();
      var clientX;
      var clientY;
      if (e.type == 'touchmove'){
        clientX = e.touches[0].clientX;
        clientY = e.touches[0].clientY;
      } else if (e.type == 'mousemove'){
        clientX = e.clientX;
        clientY = e.clientY;
      }
      return {
        x: clientX - rect.left,
        y: clientY - rect.top
      };
    }

    function draw(e){
      if (!painting) return;
      context.lineWidth = 20;
      context.lineCap = "round";
      context.strokeStyle = "white";
      
      var pos = getMousePos(canvas, e)

      context.lineTo(pos.x, pos.y);
      context.stroke();
    }

    function clearCanvas(){
      context.fillStyle = "black";
      context.fillRect(0, 0, canvas.width, canvas.height);
      predDisplay.textContent="";
      confDisplay.textContent="";
    }

    function copyURL(){
      var pngURL = canvas.toDataURL();
      var inputc = document.body.appendChild(document.createElement("input"));
      inputc.value = pngURL;
      inputc.focus();
      inputc.select();
      document.execCommand('copy');
      inputc.parentNode.removeChild(inputc);
    }
    
    function getImageTensor(canvas, height, width){
      const imageTensor = tf.div(tf.browser.fromPixels(canvas, 1).resizeBilinear([height,width]),tf.scalar(255));
      return imageTensor.reshape([1, height, width, 1]);
    }
  
    function getPrediction(){
      var imgTensor = getImageTensor(canvas, 28, 28);
      const prediction = model.predict(imgTensor);
      predictedNumber = tf.argMax(prediction, 1).asScalar().dataSync()[0];
      confidence = tf.max(prediction, 1).asScalar().dataSync()[0];
      predDisplay.textContent=predictedNumber.toString();
      confDisplay.textContent=confidence.toFixed(5);
    }

    //Event listeners
    canvas.addEventListener("mousedown", startPosition);
    canvas.addEventListener("mouseup", finishPosition);
    canvas.addEventListener("mousemove", draw);
    
    canvas.addEventListener("touchstart", startPosition);
    canvas.addEventListener("touchend", finishPosition);
    canvas.addEventListener("touchmove", draw);
  
    // Prevent scrolling when touching the canvas
    document.body.addEventListener("touchstart", function (e) {
        if (e.target == canvas) {
            e.preventDefault();
        }
    }, { passive: false });
    document.body.addEventListener("touchend", function (e) {
        if (e.target == canvas) {
            e.preventDefault();
        }
    }, { passive: false });
    document.body.addEventListener("touchmove", function (e) {
        if (e.target == canvas) {
            e.preventDefault();
        }
    }, { passive: false });
    
    clearButton.addEventListener("click", clearCanvas);
    urlButton.addEventListener("click", copyURL);
    predictButton.addEventListener("click", getPrediction);
  });
</script>
