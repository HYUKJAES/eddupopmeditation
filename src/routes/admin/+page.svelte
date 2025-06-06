<script lang="ts">
  import { supabase } from '$lib/supabaseClient';
  import { onMount } from 'svelte';
  let password = '';
  let authed = false;
  let errorMsg = '';
  let loading = false;
  let reservations = [];
  const correctPass = import.meta.env.VITE_ADMIN_PASSWORD; // 올바른 ADMINPASS를 여기에 설정

  async function checkPassword() {
    errorMsg = '';
    if (password === correctPass) {
      authed = true;
      loading = true;
      // 데이터 불러오기
      const { data, error } = await supabase
        .from('class_reservations')
        .select('*')
        .order('status', { ascending: false }) 
        .order('date', { ascending: true })
        .order('time', { ascending: true });
      if (error) {
        errorMsg = '데이터를 불러오지 못했습니다.';
      } else {
        // reserved_count > 0 인 데이터만 필터링
        reservations = data.filter(r => r.reserved_count > 0);
      }
      loading = false;
    } else {
      errorMsg = '비밀번호가 올바르지 않습니다.';
    }
  }

  async function copyToClipboard(messageTemplate) {
        try {
            await navigator.clipboard.writeText(messageTemplate);
            console.log('메시지가 클립보드에 복사되었습니다!');
        } catch (err) {
            console.error('클립보드 복사 실패:', err);
        }
    }

  async function updateReservationStatus(id, status) {
    const { error } = await supabase
      .from('class_reservations')
      .update({ status: status })
      .eq('id', id);
    if (error) {
      console.error('예약 상태 업데이트에 실패했습니다.');
    } else {
      console.log('예약 상태가 성공적으로 업데이트되었습니다.');
      // 업데이트된 예약 상태로 목록을 새로 불러옵니다.
      checkPassword();
    }
  }

  function sendSMS(userphone, username) {
        const messageTemplate = `
        [이뚜's Pop Meditation] 초보 힐러의 성장과정에 기꺼이 마루타로 시간내어 주심에 진심으로 고개 숙여 감사의 인사를 전하며, 안내사항을 전달합니다. 
준비사항: 단추, 버클 없는 편안한 옷차림 
주소: 울산 중구 학성로1, 103동 3083호 (마제스타워 1차, 우정동) 8층
주차 : 정문 로비 앞쪽 도로 혹은 아파트 주차장 - 출입 시 인터폰콜`;

            // 클립보드에 메시지 복사
            copyToClipboard(messageTemplate);
            const smsUrl = `sms:${userphone}`;
            window.location.href = smsUrl;
  }
</script>

<div class="min-h-screen flex flex-col items-center justify-center bg-gray-100 text-black p-4">
  {#if !authed}
    <div class="bg-white rounded-xl shadow-lg p-8 max-w-sm w-full flex flex-col items-center">
      <h1 class="text-xl font-bold mb-4">관리자 로그인</h1>
      <input type="password" class="input input-bordered bg-gray-100 w-full mb-2" placeholder="비밀번호" bind:value={password} on:keydown={(e) => e.key === 'Enter' && checkPassword()} />
      <button class="btn btn-success w-full" on:click={checkPassword}>확인</button>
      {#if errorMsg}
        <div class="alert alert-error mt-4">{errorMsg}</div>
      {/if}
    </div>
  {:else}
    <div class="bg-white rounded-xl shadow-lg w-full max-w-2xl p-1">
      <h1 class="text-xl font-bold mb-4">예약 신청 리스트</h1>
      {#if loading}
        <div class="flex flex-col items-center justify-center py-2">
          <span class="loading loading-spinner loading-lg text-success mb-4"></span>
          <div class="text-gray-800">데이터를 불러오는 중입니다...</div>
        </div>
      {:else}
        {#if reservations.length === 0}
          <div class="text-gray-800">신청 내역이 없습니다.</div>
        {:else}
          <div class="overflow-x-auto">
            <table class="table w-full border text-sm table-compact">
              <thead class="bg-black text-white">
                <tr>
                  <th class="py-1 px-1 text-center">예약일시</th>
                  <th class="py-1 px-1 text-center">예약자/연락처</th>
                  <th class="py-1 px-1 text-center">SMS</th>
                  <th class="py-1 px-1 text-center">완료</th>
                </tr>
              </thead>
              <tbody>
                {#each reservations as r}
                  <tr class="border-b border-gray-300 hover:bg-gray-50">
                    <td class="py-1 px-1 whitespace-nowrap text-center">
                      <div class="font-semibold">{r.date}</div>
                      <div class="text-xs text-gray-500">{r.time}</div>
                    </td>
                    <td class="py-1 px-1 whitespace-nowrap text-center">
                      <div class="font-semibold">{r.name}</div>
                      <div class="text-xs text-gray-500">{r.phone}</div>
                    </td>
                    <td class="py-1 px-1 text-center">
                      <button class="btn btn-xs" on:click={() => sendSMS(r.phone, r.name)}>📩</button>
                    </td>
                    <td class="py-1 px-1 text-center">
                      {#if r.status === 'F'}
                        <button class="btn btn-xs bg-green-700 text-white w-12" on:click={() => updateReservationStatus(r.id, '')}>완료</button>
                      {:else}
                        <button class="btn btn-xs w-12" on:click={() => updateReservationStatus(r.id, 'F')}>미완료</button>
                      {/if}
                    </td>
                  </tr>
                {/each}
              </tbody>
            </table>
          </div>
        {/if}
      {/if}
    </div>
  {/if}
</div>
