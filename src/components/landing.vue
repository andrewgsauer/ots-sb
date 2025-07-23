<template>
  <div class="row justify-content-center">
    <div class="col-12 col-md-8 text-center">
      <div class="landing-content">
        <h1 class="display-4 mb-4">Welcome to Pickup</h1>
        <p class="lead mb-5">
          Secure, one-time secret sharing by Sentinel Blue
        </p>
        
        <div class="info-box p-4 mb-4">
          <h3 class="h5 mb-3">How it works</h3>
          <p class="mb-0">
            Pickup allows you to share sensitive information securely. Secrets are encrypted end-to-end 
            and automatically destroyed after being accessed once, ensuring your data remains confidential.
          </p>
        </div>

        <div class="mt-5">
          <button
            class="btn btn-primary btn-lg"
            @click="$emit('navigate', '/create')"
          >
            <i class="fas fa-plus me-2" /> Create New Secret
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'AppLanding',
  
  emits: ['navigate'],
})
</script>

<style scoped>
.landing-content {
  margin-top: 2rem;
  margin-bottom: 2rem;
}

.info-box {
  background-color: rgba(255, 255, 255, 0.9);
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

[data-bs-theme="dark"] .info-box {
  background-color: rgba(52, 58, 64, 0.9);
}
</style>