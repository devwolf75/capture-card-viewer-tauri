<script>
  import { onMount } from "svelte";

  let videoValue = "null";
  let audioValue = "null";

  let videoElement;
  let audioElement;
  let aspectRatioSelect;

  let videoDevices = [];
  let audioDevices = [];

  onMount(async () => {
    videoElement = document.querySelector("video");
    audioElement = document.querySelector("audio");
    aspectRatioSelect = document.getElementById("aspectRatioSelect");

    getSources();
  });

  // Get the available video or audio sources
  async function getSources() {
    await navigator.getUserMedia(
      { audio: true, video: true },
      () => {
        navigator.mediaDevices.enumerateDevices().then((devices) => {
          videoDevices = devices.filter((device) =>
            device.kind.includes("video")
          );
          audioDevices = devices.filter((device) =>
            device.kind.includes("audio")
          );
        });
      },
      () => {
        errorMsg(
          "Permissions have not been granted to use your camera and " +
            "microphone, you need to allow the page access to your devices in " +
            "order for the demo to work."
        );
      }
    );
  }

  // Change the source to preview.
  async function selectSource(source, sourceType) {
    // Stop previous streams.
    if (window.stream) {
      window.stream.getTracks().forEach((track) => {
        track.stop();
      });
    }

    let constraints = {};
    constraints[sourceType] = {
      deviceId: source,
    };

    // Create a Stream for new source.
    const stream = await navigator.mediaDevices.getUserMedia(constraints);
    try {
      if (sourceType === "video") {
        videoElement.srcObject = stream;
        videoElement.play();
      }
      if (sourceType === "audio") {
        audioElement.srcObject = stream;
        audioElement.play();
      }
    } catch (e) {
      console.log(e);
    }
  }

  function changeAspectRatio(ratio) {
    let selection = aspectRatioSelect.options[aspectRatioSelect.selectedIndex].value;
    let currentRatio = videoElement.videoWidth / videoElement.videoHeight;
    let targetRatio = eval(selection.replace(":", "/"));
    let adjustmentRatio = targetRatio / currentRatio;
    videoElement.style.webkitTransform = `scaleX(${adjustmentRatio})`;
    videoElement.style.transform = `scaleX(${adjustmentRatio})`;
  }

  function errorMsg(msg, error) {
    errorElement.innerHTML += "<p>" + msg + "</p>";
    if (typeof error !== "undefined") {
      console.error(error);
    }
  }
</script>

<svelte:head>
  <meta charset="UTF-8" />
  <title>Capture Card Viewer</title>
  <link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/bulma@0.8.0/css/bulma.min.css"
  />
</svelte:head>

<div class="content">
  <div class="main-section">
    <video src="" />
    <audio src="" />
  </div>

  <div class="button-section">
    <select
      bind:value={videoValue}
      on:change={(event) => selectSource(event.target.value, "video")}
    >
      <option value="null">Video</option>
      {#each videoDevices as video}
        <option value={video.deviceId}>
          {video.label}
        </option>
      {/each}
    </select>
    <select
      bind:value={audioValue}
      on:change={(event) => selectSource(event.target.value, "audio")}
    >
      <option value="null">Audio</option>
      {#each audioDevices as audio}
        <option value={audio.deviceId}>
          {audio.label}
        </option>
      {/each}
    </select>
    <select id="aspectRatioSelect" on:change={(event) => changeAspectRatio(event.target.value)}>
      <option value="null">Aspect Ratio</option>
      <option value="1:1">1:1</option>
      <option value="4:3">4:3</option>
      <option value="16:9">16:9</option>
      <option value="16:10">16:10</option>
      <option value="1.85:1">1.85:1</option>
      <option value="2.35:1">2.35:1</option>
      <option value="9:16">Vertical (9:16)</option>
    </select>
  </div>

  <div class="error-section" />
</div>

<style type="text/scss">
  body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
      Helvetica, Arial, sans-serif;
    padding: 0;
    margin: 0;
    height: max-content;

    .button-section {
      display: none;
    }

    :hover {
      .button-section {
        text-align: center;
        display: block;
        position: relative;
        top: -100px;
        background-color: #fff;
      }
    }
  }

  .content {
    max-height: max-content;

    .main-section {
      display: flex;
      justify-content: center;

      video {
        height: 100vh;
        max-width: unset;
        min-height: 100%;
      }
    }

    .button-section {
      display: none;
      position: relative;
      top: -100px;
      background-color: #fff0;
    }
  }
</style>
