<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>XION-X-PLAYER</title>
<style>
  body {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    background-color: #1b1b1b;
    color: #fff;
    font-family: Arial, sans-serif;
    margin: 0;
  }

  .player-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    border: 2px solid #2196f3; /* Border color updated to blue */
    padding: 20px;
    border-radius: 10px;
    background-color: #2e2e2e;
    transition: border-color 0.3s ease; /* Smooth transition */
  }

  h1 {
    margin-top: 0;
    font-size: 24px;
  }

  #file-input {
    margin: 10px 0;
    padding: 8px 12px;
    color: #fff;
    background-color: #2196f3; /* Choose file button background color */
    border: 2px solid #2196f3; /* Choose file button border color */
    border-radius: 5px;
    cursor: pointer;
    transition: all 0.3s ease;
  }

  #file-input:focus {
    outline: none;
    border-color: #ffeb3b; /* Change border color on focus */
    box-shadow: 0 0 10px #ffeb3b; /* Add glowing effect on focus */
  }

  #audio-player {
    margin: 20px 0;
  }

  #bars {
    display: flex;
    gap: 5px;
    align-items: flex-end;
  }

  .bar {
    width: 5px;
    background-color: #4caf50; /* Default bar color */
    height: 20px; /* Minimum height */
    transition: height 0.1s ease, background-color 0.2s ease;
  }
</style>
</head>
<body>

<div class="player-container" id="player-container">
  <h1>XION-X-PLAYER</h1>
  <label for="file-input" style="cursor: pointer;">Choose File</label>
  <input type="file" id="file-input" accept="audio/*" style="display: none;">
  <audio id="audio-player" controls></audio>
  <div id="bars"></div>
</div>

<script>
  const audioPlayer = document.getElementById("audio-player");
  const fileInput = document.getElementById("file-input");
  const barsContainer = document.getElementById("bars");
  const playerContainer = document.getElementById("player-container");

  // Create 20 bars for the animation
  for (let i = 0; i < 20; i++) {
    const bar = document.createElement("div");
    bar.classList.add("bar");
    barsContainer.appendChild(bar);
  }

  // Handle audio file selection
  fileInput.addEventListener("change", function() {
    const file = this.files[0];
    if (file) {
      audioPlayer.src = URL.createObjectURL(file);
      audioPlayer.play();
      initAudioAnalyzer();
    }
  });

  function initAudioAnalyzer() {
    const audioContext = new (window.AudioContext || window.webkitAudioContext)();
    const analyzer = audioContext.createAnalyser();
    const source = audioContext.createMediaElementSource(audioPlayer);
    source.connect(analyzer);
    analyzer.connect(audioContext.destination);
    analyzer.fftSize = 256; // Increased fftSize for better frequency resolution

    const bufferLength = analyzer.frequencyBinCount;
    const dataArray = new Uint8Array(bufferLength);

    function animateBars() {
      analyzer.getByteFrequencyData(dataArray);

      const bars = document.querySelectorAll(".bar");
      let averageVolume = 0;

      bars.forEach((bar, index) => {
        // Reduced sensitivity by smoothing out the values
        const scale = (dataArray[index] - 40) / 3; // Adjust the subtraction factor for less sensitivity
        bar.style.height = `${Math.max(scale + 10, 10)}px`; // Ensure the minimum height is visible
        averageVolume += scale;
      });

      averageVolume = averageVolume / bars.length;
      updateColorsBasedOnVolume(averageVolume);

      requestAnimationFrame(animateBars);
    }

    animateBars();
  }

  function updateColorsBasedOnVolume(volume) {
    // Adjust color intensity based on the average volume
    let color;
    if (volume < 5) {
      color = "#4caf50"; // Green for low volume
    } else if (volume < 15) {
      color = "#ffeb3b"; // Yellow for medium-low volume
    } else if (volume < 30) {
      color = "#ff9800"; // Orange for medium-high volume
    } else {
      color = "#f44336"; // Red for high volume
    }

    // Update border and bar colors
    playerContainer.style.borderColor = color;
    document.querySelectorAll(".bar").forEach(bar => {
      bar.style.backgroundColor = color;
    });
  }
</script>

</body>
</html>
