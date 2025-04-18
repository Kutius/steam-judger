<script setup lang="ts">
const steamId = ref('')
const isLoading = ref(false)
const router = useRouter()
const error = ref('')

// 如果后续能够正常访问Steam API，
// 可以添加一个时间限制，
// 用户只能在12:00-14:00尝试生成锐评价，
// 减少频率以确保获取Steam API不被限制。

// 后续还能添加一个计数器，
// 限制24小时内只能在有限次数内访问Steam API

// 获取当前静态时间

const staticTime = new Date().toLocaleTimeString()

// 创建一个函数，检查当前时间是否在12:00-14:00之间
function checkTime(timeString){
  const front = 12 * 60;  //24Сʱ��12��00
  const rear = 14 * 60;   //24Сʱ��14��00

  const timeParts = timeString.match(/(\d{1,2}):(\d{2})/);
  let hours = parseInt(timeParts[1], 10);
  const minutes = parseInt(timeParts[2], 10);

  const total = hours * 60 + minutes;

  return total >= front && total <= rear;
}

function extractSteamIdFromUrl(url: string) {
  // 1. https://web.xiaoheihe.cn/account/steam_personal_page/home?userid=14979940&steam_id=76561198438622263&heybox_id=14979940
  // 2. https://steamcommunity.com/profiles/76561198438622263/
  const _url = url.trim()
  const steamIdMatch = _url.match(/steam_id=(\d+)|profiles\/(\d+)/)
  if (steamIdMatch) {
    return steamIdMatch[1]! || steamIdMatch[2]!
  }
  return url
}

async function handleSubmit() {
  if (!steamId.value)
    return

  error.value = ''
  isLoading.value = true

  const _steamId = extractSteamIdFromUrl(steamId.value)

  // 添加判断：
  // 如果处于10:00-14:00，
  // 进入try语句，获取Steam API
  if (checkTime(staticTime)){
    try {
    // 调用获取/缓存Steam游戏数据API
    const response = await fetch(`/api/games/${encodeURIComponent(_steamId)}`)

    if (!response.ok) {
      const errorData = await response.text()
      throw new Error(errorData || '获取游戏数据失败')
    }

    const data = await response.json()

    // 检查游戏数量，如果为0则提示用户
    if (data.gameCount === 0) {
      error.value = '您的Steam账号游戏数据为空，或者数据服务异常，请稍后重试'
      isLoading.value = false
      return
    }

    // 跳转到结果页面，并传递steamId和dataId参数
    router.push(`/review?steamId=${encodeURIComponent(_steamId)}&dataId=${encodeURIComponent(data.dataId)}`)
  }
  catch (err: any) {
    error.value = err.message || '获取游戏数据失败，请稍后重试'
    isLoading.value = false
  }
}
else{
  error.value = '当前时间不可访问'
  isLoading.value = false
}
}
</script>

<template>
  <div class="mx-auto px-4 max-w-md w-full sm:px-0">
    <div class="mb-4 text-left sm:mb-6">
      <label for="steam-id" class="text-sm text-gray-700 font-medium mb-2 block dark:text-gray-200">
        <span class="flex gap-1 items-center">
          <span class="i-carbon-game-console text-lg" />
          Steam ID
        </span>
      </label>
      <div class="relative">
        <input
          id="steam-id"
          v-model="steamId"
          type="text"
          placeholder="输入 Steam ID / 主页链接"
          class="outline-none text-sm px-3 py-2.5 border border-gray-200 rounded-lg bg-white w-full transition sm:text-base sm:px-4 sm:py-3 dark:border-gray-700 focus:border-transparent dark:bg-gray-800 focus:ring-2 focus:ring-blue-500"
          :disabled="isLoading"
          @keyup.enter="handleSubmit"
        >
      </div>
      <div class="text-xs text-gray-500 mt-2 flex gap-1 items-center dark:text-gray-400">
        <span class="i-carbon-information" />
        <a href="https://help.steampowered.com/zh-cn/faqs/view/2816-BE67-5B69-0FEC" target="_blank" class="text-blue-500 hover:underline">如何查找我的Steam ID?</a>
      </div>

      <!-- 错误信息显示 -->
      <div v-if="error" class="text-sm text-red-500 mt-2 flex gap-1 items-center">
        <span class="i-carbon-warning" />
        <span>{{ error }}</span>
      </div>
    </div>

    <button
      class="text-sm text-white font-medium px-4 py-2.5 rounded-lg bg-blue-600 flex gap-2 w-full transition-colors items-center justify-center sm:text-base sm:px-6 sm:py-3 hover:bg-blue-700"
      :class="{ 'opacity-70 cursor-not-allowed': isLoading }"
      :disabled="isLoading || !steamId"
      @click="handleSubmit"
    >
      <span v-if="isLoading" class="i-carbon-circle-dash animate-spin" />
      <span v-else class="i-carbon-analytics" />
      {{ isLoading ? '生成中...' : '生成AI锐评' }}
    </button>
    <!-- <button
      class="text-sm text-white font-medium px-4 py-2.5 rounded-lg bg-blue-600 flex gap-2 w-full transition-colors items-center justify-center sm:text-base sm:px-6 sm:py-3 hover:bg-blue-700"
      :class="{ 'opacity-70 cursor-not-allowed': isLoading }"
      :disabled="isLoading || !steamId"
    >
     Steam 数据接口被限制访问 正在寻求解决方案
    </button> -->
  </div>
</template>
