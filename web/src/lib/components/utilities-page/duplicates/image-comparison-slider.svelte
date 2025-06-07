<script lang="ts">
  interface Props {
    leftImage: string;
    rightImage: string;
    leftAlt?: string;
    rightAlt?: string;
  }
  let { leftImage, rightImage, leftAlt = '', rightAlt = '' }: Props = $props();

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
  <img class="comparison-image" src={rightImage} alt={rightAlt} draggable="false" />
  <img
    class="comparison-image"
    src={leftImage}
    alt={leftAlt}
    draggable="false"
    style="clip-path: inset(0 {100 - slider}% 0 0);"
  />
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
    cursor: ew-resize;
    z-index: 2;
  }
</style>
