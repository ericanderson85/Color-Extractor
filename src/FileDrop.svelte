<script>
import {
  writable
} from 'svelte/store';
import {
  createEventDispatcher
} from 'svelte';
const dispatch = createEventDispatcher();
let uploadedImageUrl = writable(null);

// Handles image file upload and starts the color extraction process
async function handleImageUpload(event) {
  const file = event.target.files[0];
  if (file) {
    const dataUrl = await fileToDataUrl(file);
    uploadedImageUrl.set(dataUrl);
    const imageElement = createImageElement(dataUrl);
    imageElement.onload = () => {
      dispatch('imageReady', {
        imageElement
      });
    };
  }
}

function fileToDataUrl(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => resolve(reader.result);
    reader.onerror = reject;
    reader.readAsDataURL(file);
  });
}

// Creates and returns an HTML Image element from a given source URL
function createImageElement(source) {
  const image = new Image();
  image.src = source;
  return image;
}
</script>

<input id="upload" type="file" accept="image/*" on:change={handleImageUpload} hidden>

{#if $uploadedImageUrl}

<div class="container">
  <img src={$uploadedImageUrl} alt="Uploaded file">
</div>


<div class="new-upload">
  <label for="upload">
    Upload New Image

    <svg xmlns="http://www.w3.org/2000/svg"
  height="24px" viewBox="0 -960 960 960"
  width="24px"
  fill="var(--white)">

  <path d="M180-120q-24 0-42-18t-18-42v-600q0-24 18-42t42-18h600q24 0 42 18t18 42v600q0 24-18 42t-42 18H180Zm0-60h600v-600H180v600Zm56-97h489L578-473 446-302l-93-127-117 152Zm-56 97v-600 600Z"/>
  </svg>
  </label>
</div>




{:else}

<div class="container">
<label class="upload" for="upload">
  <svg xmlns="http://www.w3.org/2000/svg"
  height="72px" viewBox="0 -960 960 960"
  width="72px"
  fill="var(--white)">

  <path d="M180-120q-24 0-42-18t-18-42v-600q0-24 18-42t42-18h600q24 0 42 18t18 42v600q0 24-18 42t-42 18H180Zm0-60h600v-600H180v600Zm56-97h489L578-473 446-302l-93-127-117 152Zm-56 97v-600 600Z"/>
  </svg>

  <span>Upload Image</span>
</label>
</div>




{/if}






<style>
.container {
  height: 40vh;
  min-height: 250px;
  width: 40vw;
  margin-left: auto;
  margin-right: auto;
  position: relative;
  
  display: flex;
  align-items: center;
  justify-content: center;
}

label {
  cursor: pointer;

  border: 1px solid var(--transparent);

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.upload {
  height: 100%;
  width: 100%;
  border-radius: 10px;
}

.new-upload label {
  padding: 20px;
  height: fit-content;
  width: fit-content;
  margin: 20px auto;
  gap: 6px;
  border-radius: 5px;
}

img {
  height: 100%;
  width: auto;
  border: 1px solid var(--transparent);
  border-radius: 10px;
}
</style>
