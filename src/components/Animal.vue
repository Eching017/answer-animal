<template>
  <div v-if="!loading" class="carousel-container" ref="carouselContainer">
    <div
      class="carousel-wrapper"
      ref="carouselWrapper"
      @mousedown="startDrag"
      @touchstart="startDrag"
    >
      <div class="carousel">
        <div
          class="carousel-item"
          :class="{ active: index === selectedIndex }"
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
import { ref, onMounted } from "vue";
import ColorThief from "colorthief";

onMounted(async () => {
  const colorPromises = animals.value.map((animal, index) =>
    getAverageColor(animal.img)
      .then((color) => {
        animalStyles.value[index] = { backgroundColor: color };
      })
      .catch((error) => {
        console.error("Failed to extract color:", error);
        animalStyles.value[index] = { backgroundColor: "#FFCC00" };
      })
  );
  await Promise.all(colorPromises);
  loading.value = false;
});

const selectedIndex = ref(null);
const animals = ref([
  { name: "img1", img: require("../assets/IMG1.png") },
  { name: "img2", img: require("../assets/IMG2.png") },
  { name: "img3", img: require("../assets/IMG3.png") },
  { name: "img4", img: require("../assets/IMG4.png") },
  { name: "img5", img: require("../assets/IMG5.png") },
  { name: "img6", img: require("../assets/IMG6.png") },
  { name: "img7", img: require("../assets/IMG7.png") },
  { name: "img8", img: require("../assets/IMG8.png") },
]);

const animalStyles = ref([]);
const loading = ref(true);
const visibleAnimals = ref(animals.value);
const carouselWrapper = ref(null);
const carouselContainer = ref(null);

let startX = 0;
let isDragging = false;

const selectAnimal = (index) => {
  selectedIndex.value = index;
  scrollToSelectedAnimal(index);
};
const changeAnimal = (direction) => {
  const newIndex =
    (selectedIndex.value + direction + visibleAnimals.value.length) %
    visibleAnimals.value.length;
  selectedIndex.value = newIndex;
  scrollToSelectedAnimal(newIndex);
};
const getAverageColor = (imgSrc) => {
  return new Promise((resolve, reject) => {
    const img = new Image();
    img.src = imgSrc;
    img.crossOrigin = "Anonymous";

    img.onload = () => {
      const colorThief = new ColorThief();
      const color = colorThief.getColor(img);
      const rgb = `rgb(${color.join(",")})`;
      resolve(rgb);
    };

    img.onerror = (err) => {
      reject(err);
    };
  });
};
const scrollToSelectedAnimal = (index) => {
  const carouselItems =
    carouselWrapper.value.querySelectorAll(".carousel-item");
  const selectedItem = carouselItems[index];

  if (selectedItem) {
    selectedItem.scrollIntoView({
      behavior: "smooth",
      block: "center",
      inline: "center",
    });
  }
};

const startDrag = (event) => {
  isDragging = true;
  startX =
    event.type === "mousedown" ? event.clientX : event.touches[0].clientX;
  carouselContainer.value.addEventListener(
    event.type === "mousedown" ? "mousemove" : "touchmove",
    onDrag
  );
  carouselContainer.value.addEventListener(
    event.type === "mousedown" ? "mouseup" : "touchend",
    stopDrag
  );
};

const onDrag = (event) => {
  if (!isDragging) return;
  const currentX =
    event.type === "mousemove" ? event.clientX : event.touches[0].clientX;
  const diff = startX - currentX;

  // Apply drag behavior by scrolling horizontally
  carouselWrapper.value.scrollLeft += diff;
  startX = currentX;
};

const stopDrag = () => {
  isDragging = false;
  carouselContainer.value.removeEventListener("mousemove", onDrag);
  carouselContainer.value.removeEventListener("touchmove", onDrag);
  carouselContainer.value.removeEventListener("mouseup", stopDrag);
  carouselContainer.value.removeEventListener("touchend", stopDrag);
};
</script>

<style scoped>
.carousel-container {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  background: linear-gradient(
    to right,
    rgba(168, 147, 147, 1),
    rgba(245, 232, 232, 0)
  );
}

.carousel-wrapper {
  display: flex;
  width: 100%;
  overflow-x: auto;
  padding: 0 30px;
  cursor: grab;
}

.carousel-wrapper:active {
  cursor: grabbing;
}

.carousel {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  height: 150px;
}

.carousel-item {
  position: relative;
  width: 100px;
  height: 100px;
  border-radius: 50%;
  margin: 0 10px;
  transition: transform 0.3s ease;
  cursor: pointer;
  z-index: 1;
  border: 5px solid white;
  flex-shrink: 0;
}

.carousel-item.active {
  transform: scale(0.9);
  z-index: 2;
  box-shadow: 0 8px 12px rgba(0, 0, 0, 0.5);
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
  transform: scale(1.5) translateY(-20%);
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
