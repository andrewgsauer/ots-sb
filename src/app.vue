<template>
  <div id="app">
    <app-navbar
      v-model:theme="theme"
      @navigate="navigate"
    />

    <div class="container mt-4">
      <div
        v-if="error"
        class="row justify-content-center"
      >
        <div class="col-12 col-md-8">
          <div
            class="alert alert-danger"
            role="alert"
            v-html="error"
          />
        </div>
      </div>

      <div class="row">
        <div class="col">
          <router-view
            @error="displayError"
            @navigate="navigate"
          />
        </div>
      </div>

      <div
        class="row mt-4 footer-row"
      >
        <div class="col text-center py-3">
          <div class="disclaimer-text mb-3">
            <p class="mb-2">
              <strong>Do not upload Controlled Unclassified Information (CUI), Protected Health Information (PHI), Payment Card Industry (PCI) data, or any other regulated or sensitive government or client data.</strong>
            </p>
            <p class="mb-2">
              This system is designed to protect the confidentiality of uploaded information using end-to-end encryption. However, Sentinel Blue makes no warranties or guarantees, expressed or implied, regarding the security, confidentiality, integrity, or availability of the data submitted through this system.
            </p>
            <p class="mb-0">
              Use of this system is at your own risk. By proceeding, you acknowledge and accept that Sentinel Blue shall not be held liable for any loss, breach, or unauthorized disclosure of information. This tool is intended for general-purpose secure sharing and should not be used for transmitting any information subject to regulatory or contractual confidentiality requirements.
            </p>
          </div>
          <div class="footer-content d-flex align-items-center justify-content-center">
            <img
              :src="footerLogo"
              alt="Sentinel Blue"
              class="footer-logo me-3"
            >
            <span
              v-if="!customize.disablePoweredBy"
              class="mx-2"
            >
              Pickup by Sentinel Blue
            </span>
            <span
              v-for="link in customize.footerLinks"
              :key="link.url"
              class="mx-2"
            >
              <a :href="link.url">{{ link.name }}</a>
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { isNavigationFailure, NavigationFailureType } from 'vue-router'

import AppNavbar from './components/navbar.vue'
import { defineComponent } from 'vue'

export default defineComponent({
  components: { AppNavbar },

  computed: {
    footerLogo(): string {
      return this.theme === 'dark' ? 'images/darkmode.png' : 'images/lightmode.png'
    },

    isSecureEnvironment(): boolean {
      return Boolean(window.crypto.subtle)
    },

    version(): string {
      return window.version
    },
  },

  created() {
    // Don't navigate away from current route
    // Only navigate if we have a hash (legacy URL)
    if (window.location.hash) {
      this.hashLoad()
    }
  },

  data() {
    return {
      customize: {} as any,
      error: '' as string | null,
      theme: 'auto',
    }
  },

  methods: {
    displayError(error: string | null) {
      this.error = error
    },

    // hashLoad reacts on a changed window hash an starts the diplaying of the secret
    hashLoad() {
      const hash = decodeURIComponent(window.location.hash)
      if (hash.length === 0) {
        return
      }

      const parts = hash.substring(1).split('|')
      const secretId = parts[0]
      let securePassword = null as string | null

      if (parts.length === 2) {
        securePassword = parts[1]
      }

      this.navigate({
        path: '/pickup',
        query: {
          secretId,
          securePassword,
        },
      })
    },

    navigate(to: string | any): void {
      this.error = ''
      this.$router.replace(to)
        .catch(err => {
          if (isNavigationFailure(err, NavigationFailureType.duplicated)) {
            // Hide duplicate nav errors
            return
          }
          throw err
        })
    },
  },

  // Trigger initialization functions
  mounted() {
    this.customize = window.OTSCustomize

    window.onhashchange = this.hashLoad
    // Don't call hashLoad here - already handled in created() if needed

    if (!this.isSecureEnvironment) {
      this.error = this.$t('alert-insecure-environment')
    }

    this.theme = window.getThemeFromStorage()
    window.matchMedia('(prefers-color-scheme: light)')
      .addEventListener('change', () => {
        window.refreshTheme()
      })
    
    // Set background image
    const app = document.getElementById('app')
    if (app) {
      app.style.backgroundImage = 'url(/images/background.png)'
    }
  },

  name: 'App',

  watch: {
    theme(to): void {
      window.setTheme(to)
    },
  },
})
</script>
