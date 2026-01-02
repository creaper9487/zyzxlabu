<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue'

const trackRef = ref(null)
const progress = ref(0)

// 處理捲動邏輯
const handleScroll = () => {
  if (!trackRef.value) return

  // 取得外層軌道相對於視窗的位置
  const rect = trackRef.value.getBoundingClientRect()
  
  // 軌道的總高度
  const trackHeight = rect.height
  // 視窗高度
  const viewportHeight = window.innerHeight
  
  // 計算有效捲動距離 (總高度 - 視窗高度)
  // 當 rect.top 為 0 時，剛好頂部碰到視窗
  // 當 rect.top 為 -(trackHeight - viewportHeight) 時，底部剛好離開視窗
  const scrollDist = trackHeight - viewportHeight
  
  // 計算目前捲了多少 (轉為正數)
  const scrolled = -rect.top
  
  // 計算百分比 (0 ~ 1)
  let p = scrolled / scrollDist
  
  // 限制範圍在 0 到 1 之間
  if (p < 0) p = 0
  if (p > 1) p = 1
  
  progress.value = p
}

onMounted(() => {
  window.addEventListener('scroll', handleScroll, { passive: true })
  // 初始化一次
  handleScroll()
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
})

// 計算橫向位移
// 總寬度是 200vw，視窗是 100vw，所以需要移動的距離是 100vw (即 window.innerWidth)
const transformStyle = computed(() => {
  // 為了 SSR 安全，這裡先給預設值，mounted 後會修正
  const moveDistance = typeof window !== 'undefined' ? window.innerWidth : 0
  const x = progress.value * moveDistance
  return {
    transform: `translate3d(-${x}px, 0, 0)`
  }
})
</script>

<template>
  <div class="relative w-full">
    <!-- 
      核心組件軌道 (Track)
      h-[300vh]: 這裡的高度決定了橫向捲動的「速度」。
      越高代表使用者要捲越久才會走完橫向流程 (感覺越慢/越平滑)。
    -->
    <div ref="trackRef" class="relative h-[300vh]">
      
      <!-- 
        Sticky 容器 
        sticky top-0: 當軌道頂部碰到視窗頂部時，這個 div 會黏住不動，
        直到軌道底部到達視窗底部才會被推走。
      -->
      <div class="sticky top-0 h-screen w-screen overflow-hidden bg-slate-600">
        
        <!-- 橫向移動層 -->
        <div 
          class="flex flex-row w-[200vw] h-full will-change-transform"
          :style="transformStyle"
        >
          
          <!-- 第一頁: 白色區塊 -->
          <div class="w-screen h-full bg-white shrink-0 flex items-center justify-center relative">
             <!-- <p class="text-9xl font-black text-slate-100 absolute select-none">01</p>
             <div class="z-10 text-center">
               <h2 class="text-3xl font-bold">歡迎來到 Sea Wallet</h2>
               <p class="text-gray-500 mt-2">請繼續向下捲動以查看更多 &rarr;</p>
             </div> -->
          </div>
          
          <!-- 第二頁: 你的主要內容 -->
          <div class="w-screen h-full flex shrink-0">
            
            <!-- 文字區 (1/3) -->
            <div class="w-1/3 h-full bg-slate-50 flex flex-col justify-center px-12 border-r border-slate-200 relative">
               <p class="text-9xl font-black text-slate-200 absolute top-10 left-10 select-none -z-0">02</p>
               <div class="z-10">
                 <h2 class="text-4xl font-bold underline decoration-indigo-500 decoration-4 text-slate-800">
                   Sea Wallet
                 </h2>
                 <p class="mt-8 text-lg text-slate-600 leading-relaxed">
                   Store, inherit, utilize your asset with the most intelligent smart contract wallet 
                   on Sui network.
                 </p>
                 <button class="mt-8 px-8 py-3 bg-indigo-600 text-white font-medium rounded-full hover:bg-indigo-700 transition shadow-lg">
                   Explore Now
                 </button>
               </div>
            </div>
            
            <!-- 圖片區 (2/3) -->
            <div class="w-2/3 h-full overflow-hidden relative bg-slate-800">
               <!-- 這裡放你的圖片，我用 div 模擬圖片佔位 -->
               <img 
                 src="/Gemini_Generated_Image_7comoc7comoc7com.png" 
                 class="w-full h-full object-cover opacity-90 hover:scale-105 transition duration-700 ease-out" 
                 alt="App Preview"
               />
            </div>

          </div>

        </div>
      </div>
    </div>

  </div>
</template>

<style scoped>
/* 為了讓捲動更平滑，可以加上這個設定。
  但在某些瀏覽器可能會導致 position: sticky 失效，如果失效請移除。
*/
html {
  scroll-behavior: smooth;
}
</style>