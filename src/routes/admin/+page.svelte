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

  async function sendSMS(phone: string, name: string) {
    // 실제로는 백엔드 API를 만들어 호출해야 합니다.
    // 예시: /api/send-sms 엔드포인트로 POST 요청
    const message = `[EDDU] ${name}님, 싱잉볼 클래스 예약이 확정되었습니다. 문의: 010-1234-5678`;
    try {
      const res = await fetch('/api/send-sms', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ phone, message })
      });
      if (res.ok) {
        alert('문자가 전송되었습니다.');
      } else {
        alert('문자 전송에 실패했습니다.');
      }
    } catch (e) {
      alert('문자 전송 중 오류가 발생했습니다.');
    }
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
    <div class="bg-white rounded-xl shadow-lg w-full max-w-2xl p-4">
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
            <table class="table w-full border text-sm">
              <thead class="bg-black text-white">
                <tr>
                  <th class="px-4 py-3 text-center">예약일시</th>
                  <th class="px-4 py-3 text-center">예약자/연락처</th>
                  <th class="px-4 py-3 text-center">SMS</th>
                </tr>
              </thead>
              <tbody>
                {#each reservations as r}
                  <tr class="border-b border-gray-300 hover:bg-gray-50">
                    <td class="px-4 py-2 whitespace-nowrap text-center">
                      <div class="font-semibold">{r.date}</div>
                      <div class="text-xs text-gray-500">{r.time}</div>
                    </td>
                    <td class="px-4 py-2 whitespace-nowrap text-center">
                      <div class="font-semibold">{r.name}</div>
                      <div class="text-xs text-gray-500">{r.phone}</div>
                    </td>
                    <td>
                      <button class="btn btn-xs" on:click={() => sendSMS(r.phone, r.name)}>📩</button>
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
