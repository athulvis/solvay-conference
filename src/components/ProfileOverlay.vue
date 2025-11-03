<template>
  <div class="overlay" @click.self="$emit('close')">
    <div class="overlay-content">
      <button class="close-btn" @click="$emit('close')">&times;</button>
      <h2>{{ person.name }}</h2>
      <img :src="person.photo" :alt="person.name" class="profile-photo">
      <p>{{ person.bio }}</p>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    person: {
      type: Object,
      required: true
    }
  }
};
</script>

<style scoped>
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.overlay-content {
  background: white;
  padding: 30px;
  border-radius: 8px;
  max-width: 500px;
  width: 90%;
  text-align: center;
  font-color:rgba(32, 15, 15, 0.87);
  position: relative;
  box-shadow: 0 4px 15px rgba(0,0,0,0.5);
}

.profile-photo {
  max-width: 150px;
  width: 100%;
  height: 150px;
  border-radius: 50%;
  object-fit: cover;
  border: 3px solid #ccc;
  margin-bottom: 15px;
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 15px;
  font-size: 28px;
  font-weight: bold;
  cursor: pointer;
  border: none;
  background: none;
  color: #555;
}
</style>
