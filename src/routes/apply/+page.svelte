<script lang="ts">
  import { supabase } from '$lib/supabaseClient';
  import { onMount } from 'svelte';
  import { goto } from '$app/navigation';
  let slots = [];
  let selectedSlot = null;
  let selectedDate = '';
  let name = '';
  let phone = '';
  let loading = false;
  let initialLoading = true;
  let success = false;
  let agree = false;
  let errorMsg = '';

  function getKoreanDay(dateStr: string) {
    const days = ['일', '월', '화', '수', '목', '금', '토'];
    // 'YYYY-MM-DD'를 안전하게 파싱
    const [year, month, day] = dateStr.split('-').map(Number);
    const d = new Date(year, month - 1, day); // 월은 0부터 시작
    return days[d.getDay()];
  }

  onMount(async () => {
    initialLoading = true;
    const { data } = await supabase
      .from('class_reservations')
      .select('*')
      .gte('date', new Date().toISOString().slice(0, 10))
      .eq('reserved_count', 0);
    slots = data.reduce((acc, slot) => {
      const date = slot.date;
      if (!acc[date]) {
        acc[date] = [];
      }
      acc[date].push(slot);
      return acc;
    }, {});
    console.log('예약 가능한 슬롯:', slots);
    initialLoading = false;
  });

  function selectSlot(date, slot) {
    selectedSlot = slot;
    selectedDate = date;
  }

  async function apply() {
    loading = true;
    errorMsg = '';
    try {
      // Update: 해당 슬롯에 신청자 정보와 reserved_count 증가
      const { error: updateError } = await supabase
        .from('class_reservations')
        .update({
          name,
          phone,
          reserved_count: selectedSlot.reserved_count + 1
        })
        .eq('id', selectedSlot.id);
      if (updateError) throw updateError;
      // 성공 시 success 페이지로 이동
      goto('apply/success');
      return;
    } catch (e) {
      errorMsg = '신청 중 오류가 발생했습니다.';
    } finally {
      loading = false;
    }
  }
</script>

{#if initialLoading}
  <div class="min-h-screen flex flex-col items-center justify-center bg-gray-100">
    <span class="loading loading-spinner loading-lg text-success mb-4"></span>
    <div class="text-gray-600 text-lg">예약 가능한 일정을 불러오는 중입니다...</div>
  </div>
{:else}
<div class="p-4 bg-gray-100 text-black min-h-screen">
  <h1 class="text-xl text-black font-bold mb-4">티벳 싱잉볼 1:1 세션 신청</h1>
  <div class="text-black">
    {#each Object.keys(slots) as date (date)}
      <h2 class="font-semibold">{date} ({getKoreanDay(date)})</h2>
      <div class="flex flex-wrap mb-4">
        {#each slots[date] as slot}
          {#if slot.reserved_count < slot.max_capacity}
            <button
              class="btn m-1 bg-gray-400 border-none text-white"
              class:bg-green-500={selectedSlot && selectedSlot.id === slot.id}
              on:click={() => selectSlot(date, slot)}
            >
              {slot.time} ({slot.reserved_count}/{slot.max_capacity})
            </button>
          {/if}
        {/each}
      </div>
    {/each}
  </div>

  <!-- 선택 정보 표시 -->
  <div class="mb-4 grid grid-cols-2 gap-2 text-black">
    <div class="bg-white rounded-lg shadow p-3 text-center">
      <div class="text-xs">선택 날짜</div>
      <div class="font-bold text-base">{selectedDate ? `${selectedDate} (${getKoreanDay(selectedDate)})`: '-'}</div>
    </div>
    <div class="bg-white rounded-lg shadow p-3 text-center">
      <div class="text-xs">선택 시간</div>
      <div class="font-bold text-base">{selectedSlot ? selectedSlot.time : '-'}</div>
    </div>
  </div>

  <div class="mb-4">
    <input class="input input-bordered bg-white text-black w-full mb-2" placeholder="성함" bind:value={name} />
    <input type="tel" class="input input-bordered bg-white text-black w-full" placeholder="연락처" bind:value={phone} />
  </div>

  <!-- 개인정보 수집 동의 -->
  <div class="form-control mb-4">
    <label class="cursor-pointer label">
      <input type="checkbox" class="checkbox checkbox-success" bind:checked={agree} />
      <span class="label-text text-sm">개인정보 수집 및 이용에 동의합니다.</span>
    </label>
    <div class="text-sm text-gray-400 mx-2">
        <p>수집된 정보는 신청확정 회신 용도로 활용되며</p>
        <p>클래스 종료 후 삭제될 예정입니다.</p>
    </div>
  </div>

  <button class="btn bg-gray-800 text-white w-full" on:click={apply} disabled={loading || !selectedSlot || !name || !phone || !agree}>
    {loading ? '신청 중...' : '최종 신청하기'}
  </button>
  {#if errorMsg}
    <div class="alert alert-error mt-4">{errorMsg}</div>
  {/if}
</div>
{/if}