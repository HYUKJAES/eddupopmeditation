<script>
  import img1 from '$lib/image/1.png';
  import img2 from '$lib/image/2.png';
  import img3 from '$lib/image/3.png';
  import { onMount, onDestroy } from 'svelte';
  import { goto } from '$app/navigation';
  
  const images = [img1, img2, img3];
  let selectedIndex = 0;
  let intervalId;

  onMount(() => {
    intervalId = setInterval(() => {
      selectedIndex = (selectedIndex + 1) % images.length;
    }, 7000);
  });
  onDestroy(() => {
    clearInterval(intervalId);
  });

  function goTo(idx) {
    selectedIndex = idx;
  }
  function goApply() {
    goto('/apply');
  }
</script>

<style>
  .slider {
    display: flex;
    transition: transform 0.5s cubic-bezier(0.4, 0, 0.2, 1);
    will-change: transform;
  }
  .slide {
    min-width: 100%;
    box-sizing: border-box;
  }
</style>

<div class="min-h-screen flex flex-col items-center justify-center bg-gray-100 relative">
  <div class="w-full max-w-sm mx-auto mt-4 overflow-hidden rounded-box relative">
    <div
      class="slider"
      style="transform: translateX(-{selectedIndex * 100}%);"
    >
      {#each images as img, i}
        <div class="slide">
          <img src={img} class="w-full object-cover rounded-box" alt={`명상 이미지 ${i+1}`}/>
        </div>
      {/each}
    </div>
    <div class="absolute flex justify-between transform -translate-y-1/2 left-2 right-2 top-1/2 w-full px-2">
      <button class="btn btn-circle btn-xs" on:click={() => goTo((selectedIndex - 1 + images.length) % images.length)}>❮</button>
      <button class="btn btn-circle btn-xs" on:click={() => goTo((selectedIndex + 1) % images.length)}>❯</button>
    </div>
  </div>

  <!-- 하단 고정 신청하기 버튼 -->
  <div class="justify-center z-50">
    <button
      class="btn btn-lg rounded-full shadow-lg mb-6 py-8 w-[90vw] max-w-sm text-lg font-bold tracking-wide animate-bounce"
      on:click={goApply}
    >
      원데이 체험 신청하기
    </button>
  </div>
</div>
