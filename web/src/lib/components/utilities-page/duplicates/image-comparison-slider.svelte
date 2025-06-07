<script lang="ts">
  import { getAssetOriginalUrl } from '$lib/utils';
  import { type AssetResponseDto } from '@immich/sdk';

  interface Props {
    leftImage: AssetResponseDto;
    rightImage: AssetResponseDto;
  }
  let { leftImage, rightImage }: Props = $props();

  let slider = $state(50);
</script>

<div
  class="comparison-container pt-4 rounded-3xl border dark:border-2 border-gray-300 dark:border-gray-700 max-w-216 mb-16"
  role="slider"
  tabindex="0"
  aria-valuenow={slider}
  aria-valuemin="0"
  aria-valuemax="100"
  aria-label="Image comparison slider"
  onmousemove={(e) => {
    const container = e.currentTarget as HTMLElement;
    const rect = container.getBoundingClientRect();
    slider = Math.max(0, Math.min(100, ((e.clientX - rect.left) / rect.width) * 100));
  }}
>
  <!-- Right image (background, fully visible) -->
  <div class="comparison-image-container" style="clip-path: inset(0 0 0 {slider}%);">
    <div class="filename right-filename">{rightImage.originalFileName}</div>
    <img
      class="comparison-image"
      src={getAssetOriginalUrl(rightImage.id)}
      alt={rightImage.originalFileName}
      draggable="false"
    />
  </div>

  <!-- Left image (foreground, clipped based on slider) -->
  <div class="comparison-image-container" style="clip-path: inset(0 {100 - slider}% 0 0);">
    <div class="filename left-filename">{leftImage.originalFileName}</div>
    <img
      class="comparison-image"
      src={getAssetOriginalUrl(leftImage.id)}
      alt={leftImage.originalFileName}
      draggable="false"
    />
  </div>

  <div class="comparison-handle" style="left: {slider}%"></div>
</div>

<style>
  .comparison-container {
    position: relative;
    width: 100%;
    max-width: 1248px;
    aspect-ratio: 4/3;
    overflow: hidden;
    user-select: none;
  }
  .comparison-image-container {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
  }
  .comparison-image {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: contain;
  }
  .comparison-handle {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 2px;
    background: #fff;
    z-index: 2;
  }
  .filename {
    position: absolute;
    background-color: rgba(0, 0, 0, 0.5);
    color: white;
    padding: 4px 8px;
    margin: 10px;
    border-radius: 4px;
    font-size: 0.875rem;
    max-width: 45%;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    z-index: 1;
  }
  .left-filename {
    top: 0;
    left: 0;
  }
  .right-filename {
    top: 0;
    right: 0;
  }
</style>
