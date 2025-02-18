<template>
  <div v-if="!loading" class="carousel-container">
    <div class="carousel-wrapper" ref="carouselWrapper">
      <div class="carousel">
        <!-- 仅渲染 visibleAnimals 中的项 -->
        <div
          class="carousel-item"
          :class="{'active': index === selectedIndex}"
          v-for="(animal, index) in visibleAnimals"
          :key="index"
          @click="selectAnimal(index)"
          :style="animalStyles[index]"
        >
          <div class="animal-img-container">
            <img :src="animal.img" :alt="animal.name" class="animal-img" />
          </div>
        </div>
      </div>
    </div>
    <div class="arrow left" @click="changeAnimal(-1)">&#9664;</div>
    <div class="arrow right" @click="changeAnimal(1)">&#9654;</div>
  </div>
  <div v-else class="loading">Loading...</div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue';
import ColorThief from 'colorthief'

const selectedIndex = ref(null);
const animals = ref([
  { name: "img1", img: require('../assets/IMG1.png') },
  { name: "img2", img: require('../assets/IMG2.png') },
  { name: "img3", img: require('../assets/IMG3.png') },
  { name: "img4", img: require('../assets/IMG4.png') },
  { name: "img5", img: require('../assets/IMG5.png') },
  { name: "img6", img: require('../assets/IMG6.png') },
  { name: "img7", img: require('../assets/IMG7.png') },
  { name: "img8", img: require('../assets/IMG8.png') },
]);

const animalStyles = ref([]); // 存储每个图标的样式
const loading = ref(true); // 加载状态
const visibleAnimals = ref(animals.value); // 存储当前需要显示的图标数组
const carouselWrapper = ref(null); // 轮播容器引用

// 选择一个动物图标
const selectAnimal = (index) => {
  selectedIndex.value = index;
  scrollToSelectedAnimal(index); // 选择后滚动到该图标
};

// 更改当前选择的动物
const changeAnimal = (direction) => {
  const newIndex = (selectedIndex.value + direction + visibleAnimals.value.length) % visibleAnimals.value.length;
  selectedIndex.value = newIndex;
  scrollToSelectedAnimal(newIndex); // 改变后滚动到该图标
};

// 获取图标的平均颜色
const getAverageColor = (imgSrc) => {
  return new Promise((resolve, reject) => {
    const img = new Image();
    img.src = imgSrc;
    img.crossOrigin = "Anonymous"; // 解决 CORS 问题（如果需要）
    
    img.onload = () => {
      const colorThief = new ColorThief();
      const color = colorThief.getColor(img); // 获取主色
      const rgb = `rgb(${color.join(',')})`; // 格式化为 RGB 字符串
      resolve(rgb);
    };

    img.onerror = (err) => {
      reject(err);
    };
  });
};

// 根据屏幕宽度调整可见的动物数量
const updateVisibleAnimals = () => {
  const screenWidth = window.innerWidth;
  const animalSize = 120; // 每个动物图标的宽度 + margin
  const maxVisibleCount = Math.floor(screenWidth / animalSize);
  visibleAnimals.value = animals.value.slice(0, maxVisibleCount);
};

// 滚动到选中的动物图标
const scrollToSelectedAnimal = (index) => {
  const carouselItems = carouselWrapper.value.querySelectorAll('.carousel-item');
  const selectedItem = carouselItems[index];

  if (selectedItem) {
    selectedItem.scrollIntoView({
      behavior: 'smooth', // 平滑滚动
      block: 'center', // 滚动到视口中间
      inline: 'center' // 保证图标在水平方向居中
    });
  }
};

// 获取颜色和更新图标显示
onMounted(async () => {
  let allColorsLoaded = true;

  for (let i = 0; i < animals.value.length; i++) {
    const animal = animals.value[i];
    try {
      const color = await getAverageColor(animal.img);
      animalStyles.value[i] = { backgroundColor: color };
    } catch (error) {
      console.error("Failed to extract color:", error);
      animalStyles.value[i] = { backgroundColor: '#FFCC00' }; // 设置默认颜色
      allColorsLoaded = false;
    }
  }

  // 一旦加载完成，更新加载状态
  loading.value = !allColorsLoaded;

  // 根据当前屏幕宽度调整显示的动物数量
  updateVisibleAnimals();
});

// 监听窗口宽度变化，调整显示的动物数量
watch(() => window.innerWidth, updateVisibleAnimals);
</script>

<style scoped>
.carousel-container {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  background-color: #a89393;
}

.carousel-wrapper {
  display: flex;
  width: 100%;
  overflow-x: auto; /* 允许横向滚动 */
  padding: 0 30px; /* 给左右两边一些空隙 */
}

.carousel {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  height: 150px;
}

.carousel-item {
  position: relative;
  width: 100px; /* 固定大小 */
  height: 100px;
  border-radius: 50%;
  margin: 0 10px;
  transition: transform 0.3s ease;
  cursor: pointer;
  z-index: 1; /* 保证圆圈在图标下面 */
  border: 5px solid white;
  flex-shrink: 0; /* 禁止图标被压缩 */
}

.carousel-item.active {
  transform: scale(0.9); /* 选中时缩小圆圈 */
  z-index: 2; /* 确保选中的项处于最上层 */
}

.animal-img-container {
  width: 100%;
  height: 120%;
  position: absolute;
  top: 0;
  left: 0;
}

.animal-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.carousel-item.active .animal-img {
  transform: scale(1.5) translateY(-20%); /* 选中时放大图标并上移 */
}

.arrow {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  font-size: 30px;
  color: #333;
  cursor: pointer;
  z-index: 100;
}

.arrow.left {
  left: 10px;
  box-shadow: 0 8px 12px rgba(0, 0, 0, 0.5);
}

.arrow.right {
  right: 10px;
  box-shadow: 0 8px 12px rgba(0, 0, 0, 0.5);
}
</style>
