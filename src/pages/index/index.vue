<script setup lang="ts">
import { ref, onMounted } from 'vue';

interface ToolItem {
  id: string;
  title: string;
  desc: string;
  url: string;
  icon: string;
  category: string;
  isFavorite: boolean;
}

const categories = ref([
  {
    id: 'favorites',
    name: '我的收藏',
    tools: [] as ToolItem[],
    showMore: false
  },
  {
    id: 'tools',
    name: '实用工具',
    tools: [
      {
        id: 'search-miniapp',
        title: '小程序查询',
        desc: '通过 AppId 查询',
        url: '/pages/tools/search-miniapp-name',
        icon: '/static/index/search.png',
        category: 'tools',
        isFavorite: false
      },
      // {
      //   id: 'photo-watermark',
      //   title: '照片水印',
      //   desc: '添加专业水印',
      //   url: '/pages/tools/add-photo-mask',
      //   icon: '/static/index/watermark.png',
      //   category: 'tools',
      //   isFavorite: false
      // }
    ],
    showMore: false
  },
  {
    id: 'others',
    name: '其他',
    tools: [
      {
        id: 'other-number-calc',
        title: '数字计算',
        desc: '滑动操作快速进行数字计算',
        url: '/pages/tools/2048',
        icon: '',
        category: 'games',
        isFavorite: true
      },
      // {
      //   id: 'random-cook',
      //   title: '随机做饭',
      //   desc: '今天吃什么',
      //   url: '/pages/tools/random-cook',
      //   icon: '/static/index/food.png',
      //   category: 'games',
      //   isFavorite: false
      // },
      // {
      //   id: 'random-travel',
      //   title: '随机旅游',
      //   desc: '去哪玩好呢',
      //   url: '/pages/tools/random-travel',
      //   icon: '/static/index/travel.png',
      //   category: 'games',
      //   isFavorite: false
      // }
    ],
    showMore: false
  }
]);

// Initialize favorites from storage
const initializeFavorites = () => {
  const storedFavorites: string[] = uni.getStorageSync('favoriteTools') || [];
  
  // Update all tools' isFavorite status based on storage
  categories.value.forEach(category => {
    category.tools.forEach(tool => {
      tool.isFavorite = storedFavorites.includes(tool.id);
    });
  });
  
  // Update favorites category
  updateFavoritesCategory();
};

// Update favorites category based on current favorites
const updateFavoritesCategory = () => {
  const favoritesCategory = categories.value.find(cat => cat.id === 'favorites');
  if (!favoritesCategory) return;
  
  // Get all favorited tools from all categories except the favorites category itself
  const favoritedTools = categories.value
    .filter(cat => cat.id !== 'favorites') // Exclude the favorites category
    .flatMap(cat => cat.tools.filter(tool => tool.isFavorite))
    .sort((a, b) => a.title.localeCompare(b.title)); // Sort alphabetically
    
  console.log('favoritedTools', favoritedTools)
  favoritesCategory.tools = favoritedTools;
};

// Toggle favorite status
const toggleFavorite = (tool: ToolItem, event: Event) => {
  // Toggle the favorite status
  tool.isFavorite = !tool.isFavorite;
  
  // Get current favorites from storage
  const favorites: string[] = uni.getStorageSync('favoriteTools') || [];
  
  // Use a Set to ensure no duplicates
  const favoritesSet = new Set(favorites);
  
  if (tool.isFavorite) {
    // Add to favorites if not already in the set
    favoritesSet.add(tool.id);
  } else {
    // Remove from favorites if exists
    favoritesSet.delete(tool.id);
  }
  
  // Convert back to array and save to storage
  const updatedFavorites = Array.from(favoritesSet);
  uni.setStorageSync('favoriteTools', updatedFavorites);
  
  // Update favorites category
  updateFavoritesCategory();
  
  // Prevent event bubbling
  event?.stopPropagation();
};

// Toggle show more tools in category
const toggleShowMore = (categoryId: string) => {
  const category = categories.value.find(cat => cat.id === categoryId);
  if (category) {
    category.showMore = !category.showMore;
  }
};

// Navigate to tool
const navigateToTool = (url: string) => {
  uni.navigateTo({
    url: url
  });
};

// Initialize on component mount
initializeFavorites();

uni.showShareMenu()

// Load favorites from storage when component mounts
onMounted(() => {
  // Initialize favorites from storage
  initializeFavorites();
});
</script>

<template>
  <view class="page-wrapper">
    <scroll-view scroll-y class="scroll-view" :enhanced="true" :show-scrollbar="false">
      <!-- Favorites Section -->
      <view class="category-section" v-for="category in categories" :key="category.id">
        <view class="category-header">
          <text class="category-title">{{ category.name }}</text>
          <text 
            v-if="category.tools.length > 6" 
            class="more-button"
            @click="toggleShowMore(category.id)"
          >
            {{ category.showMore ? '收起' : '更多' }}
          </text>
        </view>
        
        <view v-if="category.tools.length > 0" class="tools-grid">
          <view 
            v-for="(tool, index) in (category.showMore ? category.tools : category.tools.slice(0, 6))" 
            :key="tool.id"
            class="tool-item"
            @click="navigateToTool(tool.url)"
          >
            <view class="tool-icon-wrapper">
              <image class="tool-icon" :src="tool.icon || '/static/index/tool.png'" mode="aspectFit"></image>
              <image 
                class="favorite-icon" 
                :src="tool.isFavorite ? '/static/index/favorited.png' : '/static/index/favorite.png'"
                mode="aspectFit"
                @click.stop="(event) => toggleFavorite(tool, event)"
              />
            </view>
            <text class="tool-title">{{ tool.title }}</text>
            <text class="tool-desc">{{ tool.desc }}</text>
          </view>
        </view>
        
        <view v-else class="empty-category">
          <text>暂无收藏，点击工具上的⭐️添加</text>
        </view>
      </view>
      
      <!-- Coming Soon -->
      <view class="coming-soon">
        <text>更多精彩内容，敬请期待</text>
      </view>
    </scroll-view>
  </view>
</template>

<style lang="scss">
.page-wrapper {
  height: 100vh;
  background-color: #f5f5f7;
  
  .scroll-view {
    height: 100%;
    padding: 20rpx 24rpx;
    box-sizing: border-box;
  }
  
  .category-section {
    margin-bottom: 32rpx;
    background: #ffffff;
    border-radius: 16rpx;
    padding: 24rpx;
    box-shadow: 0 2rpx 12rpx rgba(0, 0, 0, 0.04);
    
    .category-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20rpx;
      
      .category-title {
        font-size: 32rpx;
        font-weight: 600;
        color: #333;
      }
      
      .more-button {
        font-size: 24rpx;
        color: #007AFF;
        padding: 8rpx 16rpx;
        border-radius: 20rpx;
        background-color: rgba(0, 122, 255, 0.1);
      }
    }
    
    .tools-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 20rpx;
      
      .tool-item {
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 20rpx 0;
        border-radius: 12rpx;
        transition: all 0.2s ease;
        
        &:active {
          background-color: rgba(0, 0, 0, 0.03);
          transform: scale(0.98);
        }
        
        .tool-icon-wrapper {
          position: relative;
          width: 100rpx;
          height: 100rpx;
          margin-bottom: 12rpx;
          
          .tool-icon {
            width: 100%;
            height: 100%;
            border-radius: 20rpx;
            background: linear-gradient(135deg, #f5f7fa, #e4e8f0);
            padding: 16rpx;
            box-sizing: border-box;
          }
          
          .favorite-icon {
            position: absolute;
            top: -8rpx;
            right: -8rpx;
            width: 36rpx;
            height: 36rpx;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 2rpx 8rpx rgba(0, 0, 0, 0.1);
            
            &.favorited {
              background: #FFF8E1;
            }
          }
        }
        
        .tool-title {
          font-size: 24rpx;
          color: #333;
          margin-bottom: 4rpx;
          text-align: center;
          font-weight: 500;
          white-space: nowrap;
          overflow: hidden;
          text-overflow: ellipsis;
          width: 100%;
        }
        
        .tool-desc {
          font-size: 20rpx;
          color: #999;
          text-align: center;
          white-space: nowrap;
          overflow: hidden;
          text-overflow: ellipsis;
          width: 100%;
        }
      }
    }
    
    .empty-category {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 120rpx;
      color: #999;
      font-size: 26rpx;
    }
  }
  
  .coming-soon {
    text-align: center;
    padding: 40rpx 0;
    color: #999;
    font-size: 26rpx;
  }
}
</style>
