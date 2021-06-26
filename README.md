# Test

<canvas id="canvas"></canvas>
<p>
  <button type="button" id="clear-button">Clear Canvas</button>
  <button type="button" id="imgulr-button">Copy img URL</button>
</p>
<script>
  window.addEventListener("load", () => {
    const canvas = document.querySelector("#canvas");
    const context = canvas.getContext("2d");
    const clearButton = document.querySelector("#clear-button");
    const urlButton = document.querySelector("#imgulr-button");

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
      return {
        x: e.clientX - rect.left,
        y: e.clientY - rect.top
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

    //Event listeners
    canvas.addEventListener("mousedown", startPosition);
    canvas.addEventListener("mouseup", finishPosition);
    canvas.addEventListener("mousemove", draw);
    clearButton.addEventListener("click", clearCanvas);
    urlButton.addEventListener("click", copyURL);
  });
</script>
