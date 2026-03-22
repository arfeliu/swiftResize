<script>
  import { onMount } from 'svelte'

  let files = []
  let compressedFiles = []
  let dragActive = false
  let fileInput
  let width = null
  let height = null
  let compression = 70
  let maintainAspect = true
  let isCompressing = false
  let selectedFormat = 'image/jpeg'
  let totalFilesProcessing = 0
  let filesCompleted = 0
  let mode = false

  const formatOptions = [
    { value: 'image/jpeg', label: 'JPEG' },
    { value: 'image/png', label: 'PNG' },
    { value: 'image/webp', label: 'WebP' },
    { value: 'image/avif', label: 'AVIF' }
  ]

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
    totalFilesProcessing = newFiles.length
    filesCompleted = 0
    newFiles.forEach(file => {
      const reader = new FileReader()
      reader.onload = (event) => {
        const img = new Image()
        img.onload = () => {
          const fileObj = {
            name: file.name,
            size: file.size,
            preview: event.target.result,
            file: file,
            mimeType: file.type,
            originalWidth: img.width,
            originalHeight: img.height
          }
          // Automatically compress the file
          compressImageAndAdd(fileObj)
        }
        img.src = event.target.result
      }
      reader.readAsDataURL(file)
    })
  }

  async function compressImageAndAdd(fileObj) {
    try {
      const result = await compressImage(fileObj)
      compressedFiles = [...compressedFiles, result]
    } catch (error) {
      console.error(`Error compressing ${fileObj.name}:`, error)
    } finally {
      filesCompleted += 1
      if (filesCompleted >= totalFilesProcessing) {
        totalFilesProcessing = 0
        filesCompleted = 0
      }
    }
  }

  function removeFile(index) {
    files = files.filter((_, i) => i !== index)
  }

  function removeCompressed(index) {
    compressedFiles = compressedFiles.filter((_, i) => i !== index)
  }

  function toggleUploadClick() {
    fileInput?.click()
  }

  async function compressImage(file) {
    return new Promise((resolve, reject) => {
      try {
        const reader = new FileReader()
        reader.onerror = () => reject(new Error(`Failed to read file: ${file.name}`))
        reader.onload = (event) => {
          const img = new Image()
          img.onerror = () => reject(new Error(`Failed to load image: ${file.name}`))
          img.onload = () => {
            try {
              const canvas = document.createElement('canvas')
              let finalWidth = width || img.width
              let finalHeight = height || img.height

              if (maintainAspect && (width || height)) {
                const aspectRatio = img.width / img.height
                const targetAspect = finalWidth / finalHeight
                if (aspectRatio > targetAspect) {
                  finalHeight = Math.round(finalWidth / aspectRatio)
                } else {
                  finalWidth = Math.round(finalHeight * aspectRatio)
                }
              }

              canvas.width = finalWidth
              canvas.height = finalHeight
              const ctx = canvas.getContext('2d')
              ctx.drawImage(img, 0, 0, finalWidth, finalHeight)

              const mimeType = selectedFormat || 'image/jpeg'
              const ext = getFormatExtension(mimeType)
              const nameWithoutExt = file.name.replace(/\.[^.]+$/, '')
              canvas.toBlob(
                (blob) => {
                  if (!blob) {
                    reject(new Error(`Failed to compress: ${file.name}`))
                    return
                  }
                  const reader = new FileReader()
                  reader.onerror = () => reject(new Error(`Failed to read compressed blob: ${file.name}`))
                  reader.onload = (e) => {
                    resolve({
                      name: `${nameWithoutExt}_c.${ext}`,
                      originalSize: file.size,
                      compressedSize: blob.size,
                      preview: e.target.result,
                      blob: blob,
                      width: finalWidth,
                      height: finalHeight
                    })
                  }
                  reader.readAsDataURL(blob)
                },
                mimeType,
                (100 - compression) / 100
              )
            } catch (err) {
              reject(err)
            }
          }
          img.src = event.target.result
        }
        reader.readAsDataURL(file.file)
      } catch (err) {
        reject(err)
      }
    })
  }

  function downloadImage(file) {
    const link = document.createElement('a')
    link.href = file.preview
    link.download = file.name
    link.click()
  }

  async function downloadAll() {
    for (let i = 0; i < compressedFiles.length; i++) {
      downloadImage(compressedFiles[i])
      // Add 100ms delay between downloads to avoid browser queue limits
      if (i < compressedFiles.length - 1) {
        await new Promise(resolve => setTimeout(resolve, 100))
      }
    }
  }

  function getSizeSavings(original, compressed) {
    return (((original - compressed) / original) * 100).toFixed(1)
  }

  function getFormatExtension(mimeType) {
    const extensionMap = {
      'image/jpeg': 'jpg',
      'image/png': 'png',
      'image/webp': 'webp',
      'image/avif': 'avif'
    }
    return extensionMap[mimeType] || 'jpg'
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

  {#if totalFilesProcessing > 0}
    <div class="progress-container">
      <div class="progress-header">
        <span class="progress-text">Processing {filesCompleted} of {totalFilesProcessing}</span>
        <span class="progress-percent">{Math.round((filesCompleted / totalFilesProcessing) * 100)}%</span>
      </div>
      <div class="progress-bar">
        <div class="progress-fill" style="width: {(filesCompleted / totalFilesProcessing) * 100}%"></div>
      </div>
    </div>
  {/if}

  <button class="mode-toggle-btn" on:click={() => (mode = !mode)}>
    {mode ? '⚙️ Hide Settings' : '⚙️ Show Settings'}
  </button>

  {#if mode}
    <div class="settings-panel">
      <h3>Compression Settings</h3>
      <div class="settings-grid">
        <div class="setting">
          <label for="width">Width (px)</label>
          <input type="number" id="width" bind:value={width} min="1" placeholder="Original" />
        </div>
        <div class="setting">
          <label for="height">Height (px)</label>
          <input type="number" id="height" bind:value={height} min="1" placeholder="Original" />
        </div>
        <div class="setting full">
          <label for="compression">Compression: {compression}%</label>
          <input type="range" id="compression" bind:value={compression} min="0" max="90" />
        </div>
        <div class="setting full">
          <label class="checkbox-label">
            <input type="checkbox" bind:checked={maintainAspect} />
            Maintain Aspect Ratio
          </label>
        </div>
        <div class="setting full">
          <label for="format">Output Format</label>
          <select id="format" bind:value={selectedFormat} class="format-select">
            {#each formatOptions as option}
              <option value={option.value}>{option.label}</option>
            {/each}
          </select>
        </div>
      </div>
      <p class="info-text">💡 Images compress automatically as you upload them</p>
    </div>
  {/if}

  {#if compressedFiles.length > 0}
    <div class="compressed-files">
      <h3>Compressed Images ({compressedFiles.length})</h3>
      <div class="file-grid">
        {#each compressedFiles as file, index (index)}
          <div class="file-item">
            <div class="preview">
              <img src={file.preview} alt={file.name} />
              <button class="remove-btn" on:click={() => removeCompressed(index)} title="Remove">
                ×
              </button>
            </div>
            <div class="file-details">
              <p class="file-name">{file.name}</p>
              <p class="file-size">{(file.compressedSize / 1024).toFixed(2)} KB</p>
              <p class="file-dims">{file.width}×{file.height}</p>
              <p class="savings">
                Saved: {getSizeSavings(file.originalSize, file.compressedSize)}%
              </p>
              <button class="download-btn" on:click={() => downloadImage(file)}>
                ↓ Download
              </button>
            </div>
          </div>
        {/each}
      </div>
      <button class="download-all-btn" on:click={downloadAll}>
        ↓ Download All ({compressedFiles.length})
      </button>
      <button class="clear-all-btn" on:click={() => (compressedFiles = [])}>
        🗑 Clear All
      </button>
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

  .progress-container {
    margin-top: 24px;
    padding: 16px 24px;
    background: var(--accent-bg);
    border: 1px solid var(--accent-border);
    border-radius: 8px;
  }

  .progress-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 12px;
  }

  .progress-text {
    font-size: 13px;
    color: var(--text-h);
    font-weight: 500;
  }

  .progress-percent {
    font-size: 13px;
    color: var(--accent);
    font-weight: 600;
  }

  .progress-bar {
    width: 100%;
    height: 8px;
    background: var(--border);
    border-radius: 4px;
    overflow: hidden;
  }

  .progress-fill {
    height: 100%;
    background: var(--accent);
    transition: width 0.3s ease;
    border-radius: 4px;
  }

  .mode-toggle-btn {
    width: 100%;
    padding: 12px 20px;
    margin-top: 24px;
    background: var(--accent-bg);
    color: var(--accent);
    border: 1px solid var(--accent-border);
    border-radius: 6px;
    font-size: 14px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s ease;

    &:hover {
      background: var(--accent);
      color: white;
      box-shadow: var(--shadow);
    }
  }

  .settings-panel {
    margin-top: 32px;
    padding: 24px;
    background: var(--accent-bg);
    border: 1px solid var(--accent-border);
    border-radius: 8px;
  }

  .settings-panel h3 {
    font-size: 16px;
    color: var(--text-h);
    margin: 0 0 16px 0;
  }

  .settings-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 16px;
    margin-bottom: 20px;

    @media (max-width: 600px) {
      grid-template-columns: 1fr;
    }
  }

  .setting {
    display: flex;
    flex-direction: column;
    gap: 6px;

    &.full {
      grid-column: 1 / -1;
    }
  }

  .setting label {
    font-size: 13px;
    color: var(--text-h);
    font-weight: 500;
  }

  .setting input[type='number'],
  .setting input[type='range'] {
    padding: 8px 12px;
    border: 1px solid var(--border);
    border-radius: 6px;
    background: var(--bg);
    color: var(--text-h);
    font-size: 14px;

    &:focus {
      outline: none;
      border-color: var(--accent);
      box-shadow: 0 0 0 2px var(--accent-bg);
    }
  }

  .setting input[type='range'] {
    padding: 0;
    height: 6px;
  }

  .format-select {
    padding: 8px 12px;
    border: 1px solid var(--border);
    border-radius: 6px;
    background: var(--bg);
    color: var(--text-h);
    font-size: 14px;
    cursor: pointer;

    &:focus {
      outline: none;
      border-color: var(--accent);
      box-shadow: 0 0 0 2px var(--accent-bg);
    }
  }

  .checkbox-label {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 14px;
    cursor: pointer;

    input {
      width: 16px;
      height: 16px;
      cursor: pointer;
    }
  }

  .info-text {
    font-size: 12px;
    color: var(--text);
    margin: 12px 0 0 0;
    font-style: italic;
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

  .file-size,
  .file-dims {
    font-size: 11px;
    color: var(--text);
    margin: 2px 0 0 0;
  }

  .compressed-files {
    margin-top: 48px;
    padding-top: 32px;
    border-top: 1px solid var(--border);
  }

  .compressed-files h3 {
    font-size: 16px;
    color: var(--text-h);
    margin: 0 0 16px 0;
  }

  .download-btn {
    width: 100%;
    padding: 6px 8px;
    margin-top: 8px;
    background: var(--accent-bg);
    color: var(--accent);
    border: 1px solid var(--accent-border);
    border-radius: 4px;
    font-size: 12px;
    cursor: pointer;
    transition: all 0.2s ease;

    &:hover {
      background: var(--accent);
      color: white;
    }
  }

  .download-all-btn {
    width: 100%;
    padding: 12px 20px;
    margin-top: 24px;
    background: var(--accent);
    color: white;
    border: none;
    border-radius: 6px;
    font-size: 14px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s ease;

    &:hover {
      opacity: 0.9;
      box-shadow: var(--shadow);
    }
  }

  .clear-all-btn {
    width: 100%;
    padding: 12px 20px;
    margin-top: 12px;
    background: var(--accent-bg);
    color: var(--accent);
    border: 1px solid var(--accent-border);
    border-radius: 6px;
    font-size: 14px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s ease;

    &:hover {
      background: var(--accent);
      color: white;
      box-shadow: var(--shadow);
    }
  }

  .savings {
    font-size: 10px;
    color: #10b981;
    margin: 4px 0 0 0;
    font-weight: 500;
  }
</style>
