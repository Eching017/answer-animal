<template>
  <div class="container">
    <div
      class="box"
      v-for="(box, index) in boxes" :key="index"
    >
      <div class="content">
        <i :class="box.icon"></i>
        <p>{{ box.text }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      boxes: [
        { text: "Home", icon: "fas fa-home" },
        { text: "User", icon: "fas fa-user" },
        { text: "Gear", icon: "fas fa-cog" },
        { text: "Bell", icon: "fas fa-bell" },
        { text: "Heart", icon: "fas fa-heart" },
      ]
    }
  }
}
</script>

<style scoped>
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto auto;
  gap: 10px;
  width: 90%;
  margin: 10px;
  margin-top: 20px;
}

.box {
  border: 1px solid #D3D3D3;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 10px;
  box-sizing: border-box;
  text-align: center;
  transition: box-shadow 0.3s ease;
}

.box .content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.box:hover {
  box-shadow: 1px 3px 0 rgba(0, 0, 0, 0.05), 0 2px 3px 0 rgba(0, 0, 0, 0.03), 0 0 0 1px #E0E0E0;
}

.box i {
  font-size: 30px;
  color: #666b6f;
  margin-bottom: 1px;
}

p {
  font-size: 14px;
  color: #666b6f;
}
</style>
