<script>
  import { onMount } from 'svelte'

  let files = []
  let dragActive = false
  let fileInput

  onMount(() => {
    // Add drag and drop event listeners
    const dropZone = document.querySelector('.upload-zone')
    if (dropZone) {
      dropZone.addEventListener('dragover', handleDragOver)
      dropZone.addEventListener('dragleave', handleDragLeave)
      dropZone.addEventListener('drop', handleDrop)
    }

    return () => {
      if (dropZone) {
        dropZone.removeEventListener('dragover', handleDragOver)
        dropZone.removeEventListener('dragleave', handleDragLeave)
        dropZone.removeEventListener('drop', handleDrop)
      }
    }
  })

  function handleDragOver(e) {
    e.preventDefault()
    e.stopPropagation()
    dragActive = true
  }

  function handleDragLeave(e) {
    e.preventDefault()
    e.stopPropagation()
    dragActive = false
  }

  function handleDrop(e) {
    e.preventDefault()
    e.stopPropagation()
    dragActive = false

    const droppedFiles = Array.from(e.dataTransfer.files).filter(file =>
      file.type.startsWith('image/')
    )

    addFiles(droppedFiles)
  }

  function handleFileInput(e) {
    const selectedFiles = Array.from(e.target.files)
    addFiles(selectedFiles)
  }

  function addFiles(newFiles) {
    newFiles.forEach(file => {
      const reader = new FileReader()
      reader.onload = (event) => {
        files = [
          ...files,
          {
            name: file.name,
            size: file.size,
            preview: event.target.result,
            file: file
          }
        ]
      }
      reader.readAsDataURL(file)
    })
  }

  function removeFile(index) {
    files = files.filter((_, i) => i !== index)
  }

  function toggleUploadClick() {
    fileInput?.click()
  }
</script>

<section class="upload-container">
  <div class="upload-zone" class:drag-active={dragActive} on:click={toggleUploadClick}>
    <svg class="upload-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor">
      <path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4" />
      <polyline points="17 8 12 3 7 8" />
      <line x1="12" y1="3" x2="12" y2="15" />
    </svg>
    <h2>Upload Images</h2>
    <p>Drag and drop your images here, or click to select</p>
    <p class="file-info">Supported formats: JPG, PNG, WebP, GIF</p>
  </div>

  <input
    bind:this={fileInput}
    type="file"
    multiple
    accept="image/*"
    on:change={handleFileInput}
    class="hidden-input"
  />

  {#if files.length > 0}
    <div class="uploaded-files">
      <h3>Uploaded Images ({files.length})</h3>
      <div class="file-grid">
        {#each files as file, index (index)}
          <div class="file-item">
            <div class="preview">
              <img src={file.preview} alt={file.name} />
              <button class="remove-btn" on:click={() => removeFile(index)} title="Remove">
                ×
              </button>
            </div>
            <div class="file-details">
              <p class="file-name">{file.name}</p>
              <p class="file-size">{(file.size / 1024).toFixed(2)} KB</p>
            </div>
          </div>
        {/each}
      </div>
    </div>
  {/if}
</section>

<style>
  .upload-container {
    width: 100%;
    max-width: 600px;
    margin: 0 auto;
  }

  .upload-zone {
    border: 2px dashed var(--border);
    border-radius: 8px;
    padding: 40px 20px;
    text-align: center;
    cursor: pointer;
    transition: all 0.3s ease;
    background: var(--accent-bg);

    &:hover {
      border-color: var(--accent);
      background: rgba(170, 59, 255, 0.15);
    }

    &.drag-active {
      border-color: var(--accent);
      background: var(--accent-bg);
      transform: scale(1.02);
    }
  }

  .upload-icon {
    width: 48px;
    height: 48px;
    color: var(--accent);
    margin-bottom: 16px;
    stroke-width: 1.5;
  }

  .upload-zone h2 {
    margin: 0 0 8px 0;
    color: var(--text-h);
    font-size: 20px;
  }

  .upload-zone p {
    margin: 4px 0;
    color: var(--text);
    font-size: 14px;

    &.file-info {
      color: var(--text);
      font-size: 12px;
    }
  }

  .hidden-input {
    display: none;
  }

  .uploaded-files {
    margin-top: 32px;
  }

  .uploaded-files h3 {
    font-size: 16px;
    color: var(--text-h);
    margin: 0 0 16px 0;
  }

  .file-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
    gap: 16px;
  }

  .file-item {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }

  .preview {
    position: relative;
    width: 100%;
    aspect-ratio: 1;
    border-radius: 8px;
    overflow: hidden;
    background: var(--code-bg);
    border: 1px solid var(--border);

    img {
      width: 100%;
      height: 100%;
      object-fit: cover;
    }

    &:hover .remove-btn {
      opacity: 1;
    }
  }

  .remove-btn {
    position: absolute;
    top: 4px;
    right: 4px;
    background: rgba(0, 0, 0, 0.6);
    color: white;
    border: none;
    border-radius: 50%;
    width: 28px;
    height: 28px;
    font-size: 20px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    opacity: 0;
    transition: opacity 0.2s ease;

    &:hover {
      background: rgba(0, 0, 0, 0.8);
    }
  }

  .file-details {
    padding: 0 4px;
  }

  .file-name {
    font-size: 12px;
    color: var(--text-h);
    margin: 0;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .file-size {
    font-size: 11px;
    color: var(--text);
    margin: 4px 0 0 0;
  }
</style>
