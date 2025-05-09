<script>
    import { goto } from '$app/navigation';
    function goHome() {
      goto('/');
    }
  </script>
  
  <div class="min-h-screen flex flex-col items-center justify-center bg-gray-100 text-black p-6">
    <div class="bg-white rounded-xl shadow-lg p-8 max-w-sm w-full flex flex-col items-center">
      <svg class="w-16 h-16 mb-4 text-green-400" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" /></svg>
      <h1 class="text-2xl font-bold mb-2 text-center">예약 신청이 완료되었습니다!</h1>
      <p class="text-gray-700 text-center mb-4">
        빠른 시일 내에 예약 확정 문자가<br />입력하신 연락처로 발송될 예정입니다.<br />
        감사합니다.
      </p>
      <button class="btn btn-success w-full mt-2" on:click={goHome}>홈으로 돌아가기</button>
    </div>
  </div> 