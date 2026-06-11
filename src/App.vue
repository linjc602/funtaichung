<script setup>
import { ref, computed } from 'vue';

// 移除未使用的 HelloWorld import
// import HelloWorld from './components/HelloWorld.vue'

const view = ref('home'); // home, category, detail, itinerary
const selectedCategory = ref(null);
const selectedSpot = ref(null);
const slider = ref(null);
const itinerary = ref([]);

// 行程規劃設定
const startTime = "09:00";
const stayDuration = 90; // 停留 90 分鐘
const travelDuration = 30; // 交通 30 分鐘
const areaPriority = { '市區': 1, '郊區': 2, '海線': 3, '山線': 4 };

// 一鍵優化路線：依地區排序
const optimizeRoute = () => {
  itinerary.value.sort((a, b) => {
    return (areaPriority[a.area] || 99) - (areaPriority[b.area] || 99);
  });
};

// 計算屬性：生成帶有時間戳的行程
const plannedItinerary = computed(() => {
  if (itinerary.value.length === 0) return [];
  
  let current = new Date(`2024-01-01 ${startTime}`);
  
  return itinerary.value.map((spot, index) => {
    const arrival = new Date(current);
    const departure = new Date(arrival.getTime() + stayDuration * 60000);
    
    // 為下一站準備時間 (加上交通)
    if (index < itinerary.value.length - 1) {
      current = new Date(departure.getTime() + travelDuration * 60000);
    }

    return {
      ...spot,
      arrivalTime: arrival.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit', hour12: false }),
      departureTime: departure.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit', hour12: false }),
      isLast: index === itinerary.value.length - 1
    };
  });
});

const totalSpotsCount = computed(() => itinerary.value.length);
const estimatedTotalTime = computed(() => {
  if (itinerary.value.length === 0) return "0";
  const totalMinutes = (itinerary.value.length * stayDuration) + ((itinerary.value.length - 1) * travelDuration);
  return (totalMinutes / 60).toFixed(1);
});

// 手動調整順序
const moveItem = (index, direction) => {
  const newIndex = index + direction;
  if (newIndex < 0 || newIndex >= itinerary.value.length) return;
  
  const temp = itinerary.value[index];
  itinerary.value[index] = itinerary.value[newIndex];
  itinerary.value[newIndex] = temp;
};

const toggleItinerary = (spot) => { // 加入或移除景點到行程
  const index = itinerary.value.findIndex(s => s.name === spot.name);
  if (index > -1) itinerary.value.splice(index, 1);
  else itinerary.value.push(spot);
};

const isInItinerary = (name) => itinerary.value.some(s => s.name === name);

const scrollSlider = (direction) => {
  // 景點詳細頁面的圖片輪播功能
  if (slider.value) {
    const scrollAmount = slider.value.clientWidth;
    slider.value.scrollBy({
      left: direction === 'left' ? -scrollAmount : scrollAmount,
      behavior: 'smooth'
    });
  }
};

const categories = ref([
  { 
    id: 1, 
    name: '文青特色店', 
    icon: '☕', 
    description: '探索巷弄間的獨立咖啡與選物店',
    spots: [
      { 
        name: '審計新村', 
        area: '市區', 
        address: '台中市西區民生路368巷',
        images: [
          'https://yuki.tw/wp-content/uploads/2018/09/20221028232332_70.jpg',
          'https://www.niusnews.com/upload/posts/posts_image3_111873_1633420252.jpg',
          'https://img.fun-life.com.tw/Taichung-play/audit-new-village/1.jpg'
        ],
        desc: `原為省政府時期審計議題之員工宿舍，如今轉型為文創聚落。這裡聚集了許多特色選物店與手作甜點，充滿年輕創創氣息，是台中最熱門的文青聚集地。`,
        foodRecommendations: [
          {
            name: '旅禾泡芙之家',
            image: 'https://dina.tw/wp-content/uploads/2024/10/01%E6%97%85%E7%A6%BE%E6%B3%A1%E8%8A%99.jpg',
            intro: '園區超人氣甜點名店，販售多種爆餡泡芙，外酥內餡飽滿。店門口設有大型摩艾石像與特色裝置藝術，是必拍的打卡地標。'
          },
          {
            name: '成真咖啡（Come True Coffee）',
            image: 'https://custom-images.strikinglycdn.com/res/hrscywv4p/image/upload/c_limit,fl_lossy,h_9000,w_1200,f_auto,q_auto/1311618/958363_735365.jpeg',
            intro: '品牌創始店之一，提供冠軍咖啡師的創意咖啡與人氣舒芙蕾鬆餅。店內環境舒適且充滿設計感，適合作為逛街後休息聊天的場所。'
          },
          {
            name: '魚刺人雞蛋糕',
            image: 'https://flyblog.cc/wp-content/uploads/2023/07/%E5%B0%81%E9%9D%A2-3.jpg',
            intro: '由偉士牌餐車起家，以酥脆外皮和濃郁蛋香聞名。除了經典口味，也推出結合台灣茶香的特色雞蛋糕，是必買的人氣點心。'
          }
        ]
      },
      { 
        name: '綠光計畫', 
        area: '市區', 
        address: '台中市西區中興一巷19號',
        images: ['https://greenripple.com.tw/app/uploads/%E7%B6%A0%E5%85%89%E8%A8%88%E5%8A%83%E5%91%A8%E9%82%8A.jpg'],
        desc: '范特喜微創文化的代表作，將舊自來水公司宿舍修築成文創聚落。保留了老建築的斑駁美感，結合藝術、設計與美食，讓人彷彿置身於時光廊道。'
      },
      { 
        name: '文化資產園區', 
        area: '市區', 
        address: '台中市南區復興路三段362號',
        images: ['https://travelblog.tw/wp-content/uploads/2024/05/20240526212801_0_71a52c.jpg'],
        desc: '前身為台中舊酒廠，是全台保存最完整的酒廠工業遺址。目前規劃為展覽與工作坊空間，結合工業遺產的粗獷與現代藝術的精緻。'
      },
      { 
        name: '國家漫畫博物館', 
        area: '市區', 
        address: '台中市西區林森路33號',
        images: ['https://shin.tw/wp-content/uploads/2024/03/ntmc-f.jpg'],
        desc: '位於台中刑務所官舍群，在古色古香的日式建築中展示台灣漫畫的發展歷史與作品。環境清幽，是漫迷與歷史愛好者的必訪點。'
      },
      { 
        name: '台中文學館', 
        area: '市區', 
        address: '台中市西區樂群街38號',
        images: [
          'https://www.tlm.taichung.gov.tw/df_ufiles/e/s_IMG_2417.jpg',
          'https://www.khotel.tw/uploads/1/2/3/4/123477094/dsc-0161_orig.jpg'
        ],
        desc: '原本是警察宿舍群，以文學為主題規劃的景點。園區內有巨大的老榕樹，木造日式建築中展出台中的文學底蘊，氛圍極其放鬆。'
      },
      { 
        name: '國立台灣美術館', 
        area: '市區', 
        address: '台中市西區五權西路一段2號',
        images: [
          'https://travel-tw.line-scdn.net/r/travel/content/2a32faad-6580-4137-a286-e38987202faft0c72df70',
          'https://suni.tw/wp-content/uploads/20200216202641_14.jpg'
        ],
        desc: '台灣唯一的國家級美術館，擁有寬闊的草坪綠地。館內典藏豐富的台灣現代藝術作品，並經常舉辦國際級展覽，是視覺與心靈的度假享受。'
      }
    ]
  },
  { 
    id: 2, 
    name: '城市探索', 
    icon: '📸', 
    description: '探索台中城市的獨特魅力',
    spots: [
      {
        name: '臺中國家歌劇院',
        area: '市區',
        address: '台中市西屯區惠來路二段101號',
        images: [
          'https://img.ltn.com.tw/Upload/house/page/2016/08/26/160826-1495-1-MKZxm.jpg'
        ],
        desc: '由世界建築大師伊東豊雄設計，以「美聲涵洞」為概念，全建築無樑柱結構，被譽為世界最難蓋的建築之一。內部曲面牆面流動感十足，頂樓還有充滿禪意的空中花園，是感受藝術與建築美學的必訪之地。'
      },
      {
        name: '秋紅谷景觀生態公園',
        area: '市區',
        address: '台中市西屯區朝富路30號',
        images: [
          'https://upload.wikimedia.org/wikipedia/commons/thumb/7/7e/2014%E7%A7%8B%E7%B4%85%E8%B0%B7%E5%85%AC%E5%9C%92%E5%A4%9C%E6%99%AF.jpg/960px-2014%E7%A7%8B%E7%B4%85%E8%B0%B7%E5%85%AC%E5%9C%92%E5%A4%9C%E6%99%AF.jpg'
        ],
        desc: '位於繁華七期重劃區的下凹式綠洲公園，不僅具備滯洪與調節氣溫功能，更提供了都市中難得的寧靜。夜晚時周邊豪宅燈火倒映在湖面，景色浪漫迷人，非常適合悠閒散步與放鬆心情。'
      },
      {
        name: '霧峰林家宮保第園區',
        area: '郊區',
        address: '台中市霧峰區民生路26號',
        images: [
          'https://museums.moc.gov.tw/Upload/FrontPhoto/1ddfce21-7606-4d24-8311-a2ffbd152cf4.jpg'
        ],
        desc: '台灣保存最完整的清代一品官宅，展現了大宅門文化的精髓。精緻的木雕、彩繪與考究的建築配置，尤其是華麗奪目的「大花廳」戲台，完美呈現了昔日家族的顯赫風華與工藝之美。'
      },
      {
        name: '草悟道',
        area: '市區',
        address: '台中市西區館前路至民生路段',
        images: [
          'https://cc.tvbs.com.tw/img/program/upload/2022/01/21/20220121162040-6dac5c62.jpg'
        ],
        desc: '以「行草」為設計概念的景觀綠園道，帶狀的綠地串聯起科博館、勤美誠品、草悟廣場至美術館。這裡充滿了悠閒的氛圍，隨處可見大型藝術裝置與文創市集，是台中最具代表性的城市綠州。'
      }
    ]
  },
  { 
    id: 3, 
    name: '台中必吃美食', 
    icon: '🍜', 
    description: '品嚐在地小吃與特色料理',
    spots: [
      {
        name: '屋馬燒肉',
        area: '市區',
        address: '台中市西區公益路111號 (公益店)',
        images: [
          'https://www.tony60533.com/wp-content/uploads/2022/10/IMG_9543.jpg',
          'https://2024-dailyview.s3.ap-northeast-1.amazonaws.com/shared_image/2025/6/e53a6d871e68a865d91b64355bbee1376d88d4e766c7db6202f55b4d003c7843.webp'
        ],
        desc: '台中燒肉界的傳奇，以高品質肉品、貼心服務和舒適環境聞名。從頂級和牛到海鮮，每一道都令人驚艷，是慶生、聚餐的首選。建議提前預約，以免向隅。'
      },
      {
        name: '輕井澤鍋物',
        area: '市區',
        address: '台中市西屯區文心路二段201號 (文心南二店)',
        images: [
          'https://www.boncity.com/s/img/infos/37071_1.jpg',
          'https://mable.tw/wp-content/uploads/2019/01/20190130215356_55.jpg'
        ],
        desc: '以日式禪風裝潢打造的火鍋店，提供多種湯底選擇與新鮮食材。無論是個人鍋還是共鍋，都能享受到精緻的用餐體驗，是台中人氣火鍋店之一。'
      },
      {
        name: '一中商圈',
        area: '市區',
        address: '台中市北區一中街',
        images: [
          'https://travel.taichung.gov.tw/image/50357/1024x768'
        ],
        desc: '台中年輕學子的聚集地，充滿各式小吃、服飾店和娛樂場所。從雞排、滷味到手搖飲，應有盡有，是體驗台中夜市文化與平價美食的好去處。'
      },
      {
        name: '大甲蔣公路夜市',
        area: '海線', // 假設為海線，若有更精確分類可調整
        address: '台中市大甲區蔣公路',
        images: [
          'https://travel.taichung.gov.tw/image/58163/1024x768'
        ],
        desc: '位於大甲鎮瀾宮旁，是當地最具代表性的夜市。除了傳統小吃，還有許多與媽祖文化相關的特色美食。逛完夜市，別忘了到鎮瀾宮參拜。'
      }
    ]
  }
]);

const selectCategory = (category) => {
  if (category.spots) {
    selectedCategory.value = category;
    view.value = 'category';
    window.scrollTo(0, 0);
  } else {
    alert(`即將開放：${category.name}`);
  }
};

const selectSpot = (spot) => {
  selectedSpot.value = spot;
  view.value = 'detail';
  window.scrollTo(0, 0);
};

const goBack = () => {
  if (view.value === 'detail') view.value = 'category';
  else if (view.value === 'itinerary') view.value = 'home';
  else view.value = 'home';
};
</script>

<template>
  <!-- 主選單介面 -->
  <div v-if="view === 'home'" class="min-h-screen flex flex-col items-center justify-center px-6 py-12">
    <!-- 標題區域 -->
    <header class="text-center mb-16 animate-fade-in">
      <h1 class="text-4xl md:text-6xl font-bold mb-4 tracking-wide text-dark-teal">
        Hello welcome Taichung！
      </h1>
      <p class="text-xl md:text-2xl opacity-90 text-dark-teal">
          你想先去哪個地方呢？
      </p>
    </header>

    <!-- 我的行程入口按鈕 -->
    <div class="mb-12">
      <button 
        @click="view = 'itinerary'"
        class="px-8 py-3 bg-white border border-[#004D4D] text-dark-teal rounded-full font-bold hover:bg-[#004D4D] hover:text-[#FDFBF7] transition-all shadow-sm flex items-center gap-2"
      >
        📂 我的行程 ({{ totalSpotsCount }})
      </button>
    </div>
    <!-- 類型選擇區域 (響應式網格) -->
    <main class="grid grid-cols-1 md:grid-cols-3 gap-8 w-full max-w-5xl">
      <div 
        v-for="category in categories" 
        :key="category.id"
        class="hover-scale border border-[#004D4D] border-opacity-20 rounded-xl p-8 text-center cursor-pointer bg-white bg-opacity-40 backdrop-blur-sm shadow-sm hover:shadow-md"
        @click="selectCategory(category)"
      >
        <div class="text-4xl mb-4">{{ category.icon }}</div>
        <h3 class="text-2xl font-semibold mb-2 text-dark-teal">{{ category.name }}</h3>
        <p class="text-sm opacity-70 text-dark-teal">{{ category.description }}</p>
      </div>
    </main>
  </div>

  <!-- 景點列表介面 -->
  <div v-else-if="view === 'category'" class="min-h-screen px-6 py-12 max-w-5xl mx-auto">
    <button @click="goBack" class="mb-8 text-dark-teal font-bold hover:underline flex items-center">
      ← 返回類型選擇
    </button>
    <h2 class="text-3xl font-bold mb-12 text-dark-teal text-center">{{ selectedCategory.name }}</h2>
    <div class="grid grid-cols-1 md:grid-cols-2 gap-8">
      <div 
        v-for="spot in selectedCategory.spots" 
        :key="spot.name"
        class="hover-scale cursor-pointer group"
        @click="selectSpot(spot)"
      >
        <div class="overflow-hidden rounded-xl shadow-sm mb-4 aspect-video relative">
          <img :src="spot.images[0]" class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-500" />
          <!-- 快速收藏圖標 -->
          <div 
            @click.stop="toggleItinerary(spot)"
            class="absolute top-3 right-3 p-2 rounded-full bg-white bg-opacity-80 backdrop-blur-sm text-dark-teal"
          >
            {{ isInItinerary(spot.name) ? '❤️' : '🤍' }}
          </div>
        </div>
        <h3 class="text-xl font-bold text-dark-teal">{{ spot.name }}</h3>
        <p class="text-sm opacity-60 text-dark-teal">{{ spot.area }}</p>
      </div>
    </div>
  </div>

  <!-- 景點詳細介紹頁面 -->
  <div v-else-if="view === 'detail'" class="min-h-screen bg-white bg-opacity-50">
    <!-- 頂端大主圖 -->
    <div class="w-full h-[50vh] md:h-[65vh] relative group">
      <div ref="slider" class="flex overflow-x-auto snap-x snap-mandatory h-full scroll-smooth scrollbar-hide">
        <div v-for="(img, index) in selectedSpot.images" :key="index" class="w-full h-full flex-shrink-0 snap-center">
          <img :src="img" class="w-full h-full object-cover animate-scale-in" />
        </div>
      </div>

      <!-- 左右導覽箭頭 -->
      <button 
        v-if="selectedSpot.images.length > 1"
        @click="scrollSlider('left')" 
        class="absolute left-4 top-1/2 -translate-y-1/2 bg-black bg-opacity-20 text-white w-10 h-10 flex items-center justify-center rounded-full hover:bg-opacity-40 backdrop-blur-sm transition-all z-10"
      >
        ‹
      </button>
      <button 
        v-if="selectedSpot.images.length > 1"
        @click="scrollSlider('right')" 
        class="absolute right-4 top-1/2 -translate-y-1/2 bg-black bg-opacity-20 text-white w-10 h-10 flex items-center justify-center rounded-full hover:bg-opacity-40 backdrop-blur-sm transition-all z-10"
      >
        ›
      </button>

      <button 
        @click="goBack" 
        class="absolute top-6 left-6 bg-black bg-opacity-30 text-white p-3 rounded-full hover:bg-opacity-50 backdrop-blur-md transition-all z-10"
      >
        ←
      </button>
      <!-- 滑動指示點 -->
      <div v-if="selectedSpot.images.length > 1" class="absolute bottom-6 left-0 right-0 flex justify-center gap-2">
        <div v-for="(_, i) in selectedSpot.images" :key="i" class="w-2 h-2 rounded-full bg-white bg-opacity-60 shadow-sm"></div>
      </div>
    </div>

    <div class="max-w-3xl mx-auto px-6 py-12">
      <div class="mb-8">
        <div class="flex flex-wrap items-center gap-4 mb-4">
          <span class="bg-[#004D4D] text-[#FDFBF7] px-3 py-1 rounded-full text-xs font-bold inline-block">
            {{ selectedSpot.area }}
          </span>
          <button 
            @click="toggleItinerary(selectedSpot)"
            class="text-sm border border-[#004D4D] px-4 py-1 rounded-full text-dark-teal hover:bg-[#004D4D] hover:text-white transition-colors"
          >
            {{ isInItinerary(selectedSpot.name) ? '❤️ 已在行程中' : '🤍 加入我的行程' }}
          </button>
        </div>
        <h1 class="text-4xl md:text-5xl font-bold text-dark-teal">{{ selectedSpot.name }}</h1>
        <!-- 地址框線區域 -->
        <div class="mt-4 border border-[#004D4D] border-opacity-40 rounded-xl px-4 py-2 inline-block text-sm text-dark-teal bg-white bg-opacity-20">
          📍 {{ selectedSpot.address }}
        </div>
      </div>

      <div class="prose lg:prose-xl mb-12">
        <p class="text-dark-teal leading-relaxed text-lg opacity-90 whitespace-pre-line">
          {{ selectedSpot.desc }}
        </p>
      </div>

      <!-- 附近美食推薦區塊 -->
      <div v-if="selectedSpot.foodRecommendations" class="mt-16 border-t border-[#004D4D] border-opacity-10 pt-12">
        <h2 class="text-2xl font-bold text-dark-teal mb-8 flex items-center gap-2">
          <span class="text-3xl">🍽️</span> 附近美食推薦
        </h2>
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
          <div 
            v-for="food in selectedSpot.foodRecommendations" 
            :key="food.name" 
            class="bg-white rounded-2xl shadow-sm overflow-hidden hover:shadow-md transition-all duration-300 border border-[#004D4D] border-opacity-5 group/food"
          >
            <div class="h-44 overflow-hidden">
              <img :src="food.image" class="w-full h-full object-cover group-hover/food:scale-110 transition-transform duration-500" />
            </div>
            <div class="p-5">
              <h3 class="font-bold text-lg text-dark-teal mb-2">{{ food.name }}</h3>
              <p class="text-sm text-dark-teal opacity-80 leading-relaxed">{{ food.intro }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- 我的行程清單介面 -->
  <div v-else-if="view === 'itinerary'" class="min-h-screen px-6 py-12 max-w-5xl mx-auto">
    <button @click="goBack" class="mb-8 text-dark-teal font-bold hover:underline flex items-center">
      ← 返回首頁
    </button>
    
    <header class="mb-12 p-8 bg-white bg-opacity-50 rounded-2xl border border-[#004D4D] border-opacity-10 shadow-sm text-center md:text-left flex flex-col md:flex-row justify-between items-center gap-6">
      <div>
        <h2 class="text-3xl font-bold text-dark-teal mb-2">我的台中漫旅行程</h2>
        <p class="text-dark-teal opacity-70 italic">慢下腳步，探索城市的美好瞬間。</p>
      </div>
      <div class="flex gap-8">
        <div class="text-center">
          <div class="text-xs opacity-60">建議出發</div>
          <div class="text-lg font-bold text-dark-teal">{{ startTime }}</div>
        </div>
        <div class="text-center">
          <div class="text-xs opacity-60">總計時長</div>
          <div class="text-lg font-bold text-dark-teal">{{ estimatedTotalTime }}h</div>
        </div>
      </div>
    </header>

    <!-- 功能按鈕 -->
    <div v-if="itinerary.length > 1" class="flex justify-end mb-8">
      <button 
        @click="optimizeRoute"
        class="px-5 py-2 bg-[#004D4D] text-[#FDFBF7] rounded-full text-sm font-bold shadow-md hover:bg-opacity-90 transition-all flex items-center gap-2"
      >
        ✨ 一鍵優化路線
      </button>
    </div>

    <!-- 時間軸行程表 -->
    <div v-if="plannedItinerary.length > 0" class="relative border-l-2 border-[#004D4D] border-opacity-20 ml-6 md:ml-12 pl-10 space-y-12">
      <div v-for="(spot, index) in plannedItinerary" :key="spot.name" class="relative">
        <!-- 時間軸圓點 -->
        <div class="absolute -left-[51px] top-0 w-5 h-5 rounded-full bg-[#004D4D] border-4 border-[#FDFBF7] shadow-sm"></div>
        
        <!-- 時間與操作 -->
        <div class="flex justify-between items-center mb-3">
          <div class="text-sm font-bold text-dark-teal opacity-60 bg-white px-3 py-1 rounded-full border border-[#004D4D] border-opacity-10">
            {{ spot.arrivalTime }} - {{ spot.departureTime }} (停留 {{ stayDuration }}m)
          </div>
          <div class="flex gap-2">
            <button @click="moveItem(index, -1)" :disabled="index === 0" class="p-1 opacity-40 hover:opacity-100 disabled:hidden">▲</button>
            <button @click="moveItem(index, 1)" :disabled="index === itinerary.length - 1" class="p-1 opacity-40 hover:opacity-100 disabled:hidden">▼</button>
          </div>
        </div>

        <!-- 景點卡片 -->
        <div class="bg-white rounded-2xl p-5 flex flex-col md:flex-row gap-5 shadow-sm border border-[#004D4D] border-opacity-5 hover:shadow-md transition-all">
          <div class="w-full md:w-32 h-24 flex-shrink-0 rounded-xl overflow-hidden">
            <img :src="spot.images[0]" class="w-full h-full object-cover" />
          </div>
          <div class="flex-grow">
            <div class="flex justify-between items-start">
              <div>
                <h3 class="font-bold text-dark-teal text-xl mb-1">{{ spot.name }}</h3>
                <p class="text-xs opacity-60">📍 {{ spot.address }}</p>
              </div>
              <button 
                @click="toggleItinerary(spot)"
                class="text-xs text-red-700 opacity-40 hover:opacity-100 underline"
              >
                移除
              </button>
            </div>
          </div>
        </div>

        <!-- 交通時間提示 -->
        <div v-if="!spot.isLast" class="absolute -left-9 -bottom-10 flex flex-col items-center opacity-30 text-dark-teal">
          <div class="text-[10px] font-black tracking-widest leading-none">🚗</div>
          <div class="text-[10px] font-bold">{{ travelDuration }}m</div>
        </div>
      </div>
    </div>

    <div v-else class="text-center py-20 bg-white bg-opacity-30 rounded-3xl border-2 border-dashed border-[#004D4D] border-opacity-10">
      <div class="text-4xl mb-4">🎒</div>
      <p class="text-dark-teal opacity-60 mb-6">您的行程還是空的，快去探索景點吧！</p>
      <button @click="view = 'home'" class="px-6 py-2 bg-[#004D4D] text-white rounded-full text-sm">
        開始探索
      </button>
    </div>
  </div>

  <div class="flex flex-col items-center">
    <footer class="mt-20 text-sm opacity-50 italic text-dark-teal">
      Enjoy your slow travel in Taichung.
    </footer>
  </div>
</template>

<style>
body {
  font-family: 'Noto Serif TC', serif;
  background-color: #FDFBF7; /* 米白色背景 */
  margin: 0;
}

.text-dark-teal {
  color: #004D4D;
}

.hover-scale {
  transition: transform 0.3s ease;
}
.hover-scale:hover {
  transform: translateY(-5px);
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
.animate-fade-in {
  animation: fadeIn 1s ease-out;
}

@keyframes scaleIn {
  from { transform: scale(1.1); opacity: 0.8; }
  to { transform: scale(1); opacity: 1; }
}
.animate-scale-in {
  animation: scaleIn 1.5s ease-out;
}

.scrollbar-hide::-webkit-scrollbar { display: none; }
.scrollbar-hide { -ms-overflow-style: none; scrollbar-width: none; }
</style>
