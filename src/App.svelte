<script>
  import { onMount } from "svelte";
  import { appWindow } from "@tauri-apps/api/window";

  let videoValue = "null";
  let audioValue = "null";

  let videoElement;
  let audioElement;
  let errorElement;
  let aspectRatioSelect;

  let videoDevices = [];
  let audioDevices = [];

  onMount(() => {
    videoElement = document.querySelector("video");
    audioElement = document.querySelector("audio");
    errorElement = document.getElementById("error-element");
    aspectRatioSelect = document.getElementById("aspectRatioSelect");

    getSources();
  });

  // Get the available video or audio sources
  async function getSources() {
    // @ts-ignore
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
      (err) => {
        errorMsg(
          "Permissions have not been granted to use your camera and " +
            "microphone, you need to allow the page access to your devices in " +
            "order for the application to work." +
            `${err}`
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
        if (source !== "null") {
          videoElement.srcObject = stream;
          videoElement.onpause = () => {
            videoElement.play();
          }; // Prevent accidental pause.
          videoElement.play();
        } else {
          videoElement.pause();
          videoElement.removeAttribute("src");
          videoElement.load();
        }
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
    let selection =
      aspectRatioSelect.options[aspectRatioSelect.selectedIndex].value;
    let currentRatio = videoElement.videoWidth / videoElement.videoHeight;
    let values = selection.split(":");
    let targetRatio = values[0] / values[1];
    let adjustmentRatio = targetRatio / currentRatio;
    videoElement.style.webkitTransform = `scaleX(${adjustmentRatio})`;
    videoElement.style.transform = `scaleX(${adjustmentRatio})`;
  }

  function makeFullScreen() {
    if (!document.fullscreenElement && videoValue !== "null") {
      videoElement.requestFullscreen();
      videoElement.removeAttribute("controls");
      appWindow.setFullscreen(true);
    } else if (document.exitFullscreen) {
      appWindow.setFullscreen(false);
      document.exitFullscreen();
    } else {
      console.error("Fullscreen API is not supported");
    }
  }

  function errorMsg(msg, error) {
    try {
      errorElement.innerHTML += "<p>" + msg + "</p>";
      if (typeof error !== "undefined") {
        console.error(error);
      }
    } catch (exception) {
      console.error(error);
    }
  }
</script>

<svelte:head>
  <meta charset="UTF-8" />
  <title>Capture Card Viewer</title>
</svelte:head>

<div class="content">
  <div class="main-section">
    <video src="" on:dblclick={(event) => makeFullScreen()} />
    <audio src="" />
  </div>

  <div class="button-section" id="button-section">
    <div
      class="select is-rounded {videoDevices.length === 0 ? 'is-loading' : ''}"
    >
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
    </div>
    <div
      class="select is-rounded {audioDevices.length === 0 ? 'is-loading' : ''}"
    >
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
    </div>
    <div class="select is-rounded">
      <select
        id="aspectRatioSelect"
        on:change={(event) => changeAspectRatio(event.target.value)}
      >
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
  </div>

  <div class="error-section" id="error-section"></div>
</div>

<style lang="scss">
  @import "https://cdn.jsdelivr.net/npm/bulma@1.0.0/css/bulma.min.css";

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
        background-color: transparent;
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

  video::-webkit-media-controls {
    display: none;
  }
</style>
