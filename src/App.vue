<template>
  <div id="app">
    <!-- Language Toggle -->
    <div class="language-toggle">
      <button
        @click="toggleLanguage"
        class="btn btn-sm btn-light">
        {{ currentLang === 'en' ? 'മലയാളം' : 'English' }}
      </button>
    </div>

    <!-- Idle Video -->
    <div v-if="showIdleVideo" class="idle-video-container" @click="stopIdleVideo">
      <video
        ref="idleVideo"
        autoplay
        loop
        muted
        class="idle-video">
        <source src="./assets/idle-video.mp4" type="video/mp4">
      </video>
    </div>

    <div v-else class="conference-photo-container">
      <img src="./assets/solvay_1927.jpg"
           alt="Solvay Conference 1927"
           ref="conferenceImage"
           @load="updateImageDimensions">

      <!-- Clickable areas trigger modal on click -->
      <div v-for="person in attendees"
         :key="person.id"
         class="clickable-area"
         :class="{ 'animate-pulse': animatingPersonId === person.id }"
         :style="getAreaStyle(person.Coords)"
         @click="handlePersonClick(person)"
         :title="currentLang === 'en' ? person.name_en : person.name">
      </div>
    </div>

    <!-- Bootstrap Modal -->
    <div v-if="selectedPerson"
         class="modal fade show d-block"
         tabindex="-1">
      <div class="modal-dialog modal-dialog-centered modal-xl">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">
              {{ currentLang === 'en' ? selectedPerson.name_en : selectedPerson.name }}
            </h5>
            <button type="button"
                    class="btn-close"
                    @click="closeModal"
                    aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <div class="row">
              <!-- Left: Photo -->
              <div class="col-md-4 text-center">
                <img :src="selectedPerson.photo"
                     :alt="currentLang === 'en' ? selectedPerson.name_en : selectedPerson.name"
                     class="profile-photo rounded-circle img-fluid">
              </div>
              <!-- Right: Bio -->
              <div class="col-md-8">
                <h4 class="mb-3">
                  {{ currentLang === 'en' ? selectedPerson.name_en : selectedPerson.name }}
                </h4>
                <p class="bio-text">
                  {{ currentLang === 'en' ? selectedPerson.bio_en : selectedPerson.bio }}
                </p>
              </div>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" @click="closeModal">
              {{ currentLang === 'en' ? 'Close' : 'അടയ്ക്കുക' }}
            </button>
          </div>
        </div>
      </div>
    </div>
    <div v-if="selectedPerson" class="modal-backdrop fade show"></div>
  </div>
</template>

<script>
import attendeesData from './assets/attendees.json';

export default {
  name: 'App',
  data() {
    return {
      attendees: attendeesData,
      selectedPerson: null,
      originalImageDimensions: { width: 3178, height: 2249 },
      currentImageDimensions: { width: 0, height: 0 },
      currentLang: 'en',
      animatingPersonId: null,
      modalAutoCloseTimer: null,
      idleTimer: null,
      showIdleVideo: false,
      lastActivityTime: Date.now()
    };
  },
  mounted() {
    window.addEventListener('resize', this.updateImageDimensions);
    window.addEventListener('keydown', this.handleKeyDown);
    this.updateImageDimensions();
    this.startIdleDetection();
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.updateImageDimensions);
    window.removeEventListener('keydown', this.handleKeyDown);
    this.clearModalAutoCloseTimer();
    this.clearIdleTimer();
  },
  methods: {
    handlePersonClick(person) {
      this.resetIdleTimer();
      this.animatingPersonId = person.id;

      // Animation duration before showing modal
      setTimeout(() => {
        this.animatingPersonId = null;
        this.showProfile(person);
      }, 600);
    },
    showProfile(person) {
      this.selectedPerson = person;
      this.startModalAutoCloseTimer();
    },
    closeModal() {
      this.selectedPerson = null;
      this.clearModalAutoCloseTimer();
    },
    updateImageDimensions() {
      this.$nextTick(() => {
        if (this.$refs.conferenceImage) {
          this.currentImageDimensions.width = this.$refs.conferenceImage.clientWidth;
          this.currentImageDimensions.height = this.$refs.conferenceImage.clientHeight;
        }
      });
    },
    getAreaStyle(coords) {
      if (!coords || this.currentImageDimensions.width === 0) {
        return { display: 'none' };
      }

      const widthRatio = this.currentImageDimensions.width / this.originalImageDimensions.width;
      const heightRatio = this.currentImageDimensions.height / this.originalImageDimensions.height;

      const scaledX = coords.x * widthRatio;
      const scaledY = coords.y * heightRatio;
      const scaledR = coords.r * widthRatio;

      return {
        position: 'absolute',
        left: `${scaledX - scaledR}px`,
        top: `${scaledY - scaledR}px`,
        width: `${scaledR * 2}px`,
        height: `${scaledR * 2}px`,
        borderRadius: '50%',
      };
    },
    handleKeyDown(event) {
      this.resetIdleTimer();
      if (event.key === 'Escape' && this.selectedPerson) {
        this.closeModal();
      }
    },
    toggleLanguage() {
      this.currentLang = this.currentLang === 'en' ? 'ml' : 'en';
      this.resetIdleTimer();
    },
    // Modal auto-close after 2 minutes
    startModalAutoCloseTimer() {
      this.clearModalAutoCloseTimer();
      this.modalAutoCloseTimer = setTimeout(() => {
        this.closeModal();
      }, 120000); // 2 minutes
    },
    clearModalAutoCloseTimer() {
      if (this.modalAutoCloseTimer) {
        clearTimeout(this.modalAutoCloseTimer);
        this.modalAutoCloseTimer = null;
      }
    },
    // Idle detection for video
    startIdleDetection() {
      this.resetIdleTimer();

      // Track user activity
      const events = ['mousedown', 'mousemove', 'keypress', 'scroll', 'touchstart', 'click'];
      events.forEach(event => {
        document.addEventListener(event, this.resetIdleTimer);
      });
    },
    resetIdleTimer() {
      this.lastActivityTime = Date.now();

      if (this.showIdleVideo) {
        return;
      }

      this.clearIdleTimer();
      this.idleTimer = setTimeout(() => {
        this.showIdleVideo = true;
      }, 300000); // 5 minutes
    },
    clearIdleTimer() {
      if (this.idleTimer) {
        clearTimeout(this.idleTimer);
        this.idleTimer = null;
      }
    },
    stopIdleVideo() {
      this.showIdleVideo = false;
      this.resetIdleTimer();
    }
  }
};
</script>

<style>
@import 'bootstrap/dist/css/bootstrap.min.css';

/* Critical: Set html and body to 100% height */
html, body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  overflow: hidden;
  background-color: #000;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
}

/* Language Toggle */
.language-toggle {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1001;
}

/* Conference Photo Container */
.conference-photo-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

/* Mobile: Full width (max-width: 767px) */
@media (max-width: 767px) {
  .conference-photo-container img {
    width: 100%;
    height: auto;
    object-fit: contain;
    display: block;
  }
}

/* Tablet: Full width (768px - 1023px) */
@media (min-width: 768px) and (max-width: 1023px) {
  .conference-photo-container img {
    width: 100%;
    height: auto;
    object-fit: contain;
    display: block;
  }
}

/* Desktop and larger: Cover full height (1024px and up) */
@media (min-width: 1024px) {
  .conference-photo-container img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
  }
}

/* Clickable area */
.clickable-area {
  position: absolute;
  cursor: pointer;
  z-index: 10;
  transition: all 0.3s ease;
}

.clickable-area:hover {
  background-color: rgba(255, 255, 255, 0.2);
  border: 2px solid rgba(255, 255, 255, 0.6);
}

/* Animation on click */
@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.3);
    opacity: 0.7;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.animate-pulse {
  animation: pulse 0.6s ease-in-out;
  background-color: rgba(255, 215, 0, 0.5);
  border: 3px solid gold;
}

/* Bootstrap Modal Styling - Larger Size */
.modal-dialog.modal-xl {
  max-width: 1000px;
  margin: 1.75rem auto;
}

/* Responsive modal */
@media (max-width: 768px) {
  .modal-dialog.modal-xl {
    max-width: 95%;
    margin: 1rem auto;
  }
}

.profile-photo {
  max-width: 250px;
  width: 100%;
  height: 250px;
  object-fit: cover;
  border: 4px solid #dee2e6;
}

.bio-text {
  font-size: 1.1rem;
  line-height: 1.8;
  text-align: justify;
}

.modal-backdrop {
  background-color: rgba(0, 0, 0, 0.8);
}

.modal-content {
  box-shadow: 0 4px 15px rgba(0,0,0,0.5);
}

/* Idle Video */
.idle-video-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 9999;
  cursor: pointer;
  background-color: #000;
}

.idle-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
</style>
