<script setup lang="ts">
import type QrScanner from 'qr-scanner'

definePageMeta({ title: 'Scan' })

const videoRef = ref<HTMLVideoElement | null>(null)
const scanning = ref(false)
const result = ref('')
const errorMsg = ref('')

let scanner: QrScanner | null = null

async function startScan() {
  errorMsg.value = ''
  result.value = ''

  if (!videoRef.value) {
    errorMsg.value = 'Camera preview is unavailable.'
    return
  }

  try {
    if (!scanner) {
      const { default: QrScannerLibrary } = await import('qr-scanner')
      scanner = new QrScannerLibrary(
        videoRef.value,
        (scanResult) => {
          result.value = scanResult.data
          stopScan()
        },
        {
          preferredCamera: 'environment',
          maxScansPerSecond: 10,
          returnDetailedScanResult: true,
        }
      )
    }

    scanning.value = true
    await scanner.start()
  } catch (err: unknown) {
    scanner?.destroy()
    scanner = null
    scanning.value = false
    errorMsg.value = err instanceof Error ? err.message : 'Unable to access camera.'
  }
}

function stopScan() {
  scanning.value = false
  scanner?.stop()
}

onBeforeUnmount(() => {
  stopScan()
  scanner?.destroy()
  scanner = null
})
</script>

<template>
  <v-container fluid class="scan-page pa-0">
    <v-row align="center" class="mb-4">
      <v-col cols="12" md="7">
        <div class="d-flex align-center">
          <v-avatar color="cyan" variant="tonal" rounded="lg" class="mr-3">
            <v-icon>mdi-qrcode-scan</v-icon>
          </v-avatar>
          <div>
            <div class="text-h4 font-weight-bold">QR Scanner</div>
            <div class="text-body-1 text-medium-emphasis">
              Scan a QR code with your device camera.
            </div>
          </div>
        </div>
      </v-col>
      <v-col cols="12" md="5" class="text-md-right">
        <v-chip
          :color="scanning ? 'teal' : 'blue'"
          variant="tonal"
          size="large"
        >
          <v-icon start size="18">
            {{ scanning ? 'mdi-radiobox-marked' : 'mdi-check-circle' }}
          </v-icon>
          {{ scanning ? 'Scanner active' : 'Ready to scan' }}
        </v-chip>
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="12" lg="8">
        <v-card rounded="xl" class="camera-card pa-4 pa-md-6" elevation="2">
          <div class="d-flex align-center justify-space-between mb-4">
            <div>
              <div class="text-h6 font-weight-bold">Camera view</div>
              <div class="text-body-2 text-medium-emphasis">
                Use your rear camera for the best results.
              </div>
            </div>
            <v-avatar color="deep-purple" variant="tonal">
              <v-icon>mdi-camera-outline</v-icon>
            </v-avatar>
          </div>

          <div class="camera-viewport" :class="{ 'camera-viewport--active': scanning }">
            <video
              ref="videoRef"
              class="camera-video"
              :class="{ 'camera-video--active': scanning }"
              muted
              playsinline
              aria-label="Live camera preview for QR scanning"
            />

            <div v-if="!scanning" class="camera-empty">
              <v-avatar color="cyan" variant="tonal" size="76" class="mb-4">
                <v-icon size="38">mdi-camera-plus-outline</v-icon>
              </v-avatar>
              <div class="text-h6 font-weight-bold">Your camera is ready</div>
              <div class="text-body-2 text-medium-emphasis mt-1">
                Start scanning and point your camera at a QR code.
              </div>
            </div>

            <div v-if="scanning" class="scan-overlay" aria-hidden="true">
              <div class="scan-target">
                <span class="scan-corner scan-corner--tl" />
                <span class="scan-corner scan-corner--tr" />
                <span class="scan-corner scan-corner--bl" />
                <span class="scan-corner scan-corner--br" />
                <span class="scan-line" />
              </div>
              <div class="scan-caption">
                <v-icon size="17" class="mr-1">mdi-focus-field</v-icon>
                Align the QR code inside the frame
              </div>
            </div>
          </div>

          <v-btn
            block
            size="large"
            :color="scanning ? 'deep-orange' : 'cyan-darken-1'"
            :prepend-icon="scanning ? 'mdi-stop-circle-outline' : 'mdi-camera'"
            class="scan-action mt-4"
            @click="scanning ? stopScan() : startScan()"
          >
            {{ scanning ? 'Stop scanning' : 'Start scanning' }}
          </v-btn>

          <v-alert
            v-if="errorMsg"
            type="error"
            variant="tonal"
            icon="mdi-camera-off"
            class="mt-4"
          >
            {{ errorMsg }}
          </v-alert>
        </v-card>
      </v-col>

      <v-col cols="12" lg="4">
        <v-card rounded="xl" class="result-card pa-5" elevation="2">
          <div class="d-flex align-center justify-space-between mb-4">
            <div class="d-flex align-center">
              <v-avatar
                :color="result ? 'teal' : 'deep-purple'"
                variant="tonal"
                class="mr-3"
              >
                <v-icon>{{ result ? 'mdi-check-decagram' : 'mdi-qrcode' }}</v-icon>
              </v-avatar>
              <div>
                <div class="text-h6 font-weight-bold">Scan result</div>
                <div class="text-body-2 text-medium-emphasis">
                  {{ result ? 'QR code detected' : 'Your result appears here' }}
                </div>
              </div>
            </div>
            <v-icon v-if="result" color="teal-lighten-1" size="24">
              mdi-check-circle
            </v-icon>
          </div>

          <v-sheet
            rounded="lg"
            class="result-content pa-4"
            :class="{ 'result-content--found': result }"
          >
            <div v-if="result" class="result-text">{{ result }}</div>
            <div v-else class="text-body-2 text-medium-emphasis">
              <v-icon size="18" class="mr-1">mdi-information-outline</v-icon>
              No code scanned yet. Start the camera to detect a QR code.
            </div>
          </v-sheet>
        </v-card>

        <v-card rounded="xl" class="tips-card pa-5 mt-4" elevation="1">
          <div class="d-flex align-center mb-4">
            <v-avatar color="amber" variant="tonal" size="36" class="mr-3">
              <v-icon size="20">mdi-lightbulb-on-outline</v-icon>
            </v-avatar>
            <div class="text-h6 font-weight-bold">Tips for scanning</div>
          </div>
          <div class="tip-row">
            <v-avatar color="cyan" variant="tonal" size="30" class="mr-3">
              <span class="text-caption font-weight-bold">1</span>
            </v-avatar>
            <span class="text-body-2">Allow camera access when prompted.</span>
          </div>
          <div class="tip-row">
            <v-avatar color="deep-purple" variant="tonal" size="30" class="mr-3">
              <span class="text-caption font-weight-bold">2</span>
            </v-avatar>
            <span class="text-body-2">Keep the QR code centered and in focus.</span>
          </div>
          <div class="tip-row">
            <v-avatar color="teal" variant="tonal" size="30" class="mr-3">
              <span class="text-caption font-weight-bold">3</span>
            </v-avatar>
            <span class="text-body-2">Use steady lighting and avoid glare.</span>
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<style scoped>
.scan-page {
  max-width: 1200px;
}

.camera-card {
  border: 1px solid rgba(var(--v-border-color), var(--v-border-opacity));
}

.camera-viewport {
  position: relative;
  display: grid;
  overflow: hidden;
  width: 100%;
  min-height: 340px;
  aspect-ratio: 4 / 3;
  place-items: center;
  border: 1px solid rgba(56, 189, 248, 0.18);
  border-radius: 18px;
  background:
    radial-gradient(ellipse at 50% 45%, rgba(45, 212, 191, 0.12), transparent 38%),
    linear-gradient(145deg, #0d172b, #101c34 58%, #13243a);
}

.camera-viewport--active {
  border-color: rgba(45, 212, 191, 0.65);
  box-shadow: inset 0 0 36px rgba(20, 184, 166, 0.12);
}

.camera-video {
  position: absolute;
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0;
  transition: opacity 200ms ease;
}

.camera-video--active {
  opacity: 1;
}

.camera-empty {
  z-index: 1;
  display: flex;
  max-width: 360px;
  flex-direction: column;
  align-items: center;
  padding: 24px;
  text-align: center;
}

.scan-overlay {
  position: absolute;
  inset: 0;
  display: grid;
  place-items: center;
}

.scan-target {
  position: relative;
  width: min(62vw, 260px);
  aspect-ratio: 1;
  border-radius: 18px;
  background: rgba(8, 15, 29, 0.12);
  box-shadow: 0 0 0 999px rgba(4, 10, 23, 0.18);
}

.scan-corner {
  position: absolute;
  width: 36px;
  height: 36px;
  border-color: #5eead4;
  border-style: solid;
  filter: drop-shadow(0 0 5px rgba(94, 234, 212, 0.65));
}

.scan-corner--tl {
  top: 0;
  left: 0;
  border-width: 4px 0 0 4px;
  border-top-left-radius: 14px;
}

.scan-corner--tr {
  top: 0;
  right: 0;
  border-width: 4px 4px 0 0;
  border-top-right-radius: 14px;
}

.scan-corner--bl {
  bottom: 0;
  left: 0;
  border-width: 0 0 4px 4px;
  border-bottom-left-radius: 14px;
}

.scan-corner--br {
  right: 0;
  bottom: 0;
  border-width: 0 4px 4px 0;
  border-bottom-right-radius: 14px;
}

.scan-line {
  position: absolute;
  top: 14%;
  right: 10px;
  left: 10px;
  height: 2px;
  background: #5eead4;
  box-shadow: 0 0 12px 2px rgba(94, 234, 212, 0.8);
  animation: scan-line 2.4s ease-in-out infinite alternate;
}

.scan-caption {
  position: absolute;
  bottom: 18px;
  display: flex;
  align-items: center;
  padding: 8px 14px;
  border: 1px solid rgba(255, 255, 255, 0.14);
  border-radius: 999px;
  color: white;
  background: rgba(9, 17, 32, 0.74);
  font-size: 0.875rem;
  backdrop-filter: blur(8px);
}

.scan-action {
  min-height: 52px;
  color: white !important;
  font-weight: 700;
  letter-spacing: 0.02em;
}

.result-card {
  border-top: 3px solid #2dd4bf;
}

.result-content {
  min-height: 112px;
  border: 1px dashed rgba(148, 163, 184, 0.28);
  background: rgba(148, 163, 184, 0.06);
}

.result-content--found {
  border-color: rgba(45, 212, 191, 0.45);
  background: rgba(45, 212, 191, 0.08);
}

.result-text {
  overflow-wrap: anywhere;
  white-space: pre-wrap;
}

.tips-card {
  border-top: 3px solid #fbbf24;
}

.tip-row {
  display: flex;
  align-items: center;
  padding: 10px 0;
}

.tip-row + .tip-row {
  border-top: 1px solid rgba(var(--v-border-color), var(--v-border-opacity));
}

@keyframes scan-line {
  from {
    top: 14%;
  }

  to {
    top: 84%;
  }
}

@media (max-width: 600px) {
  .camera-viewport {
    min-height: 280px;
  }
}

@media (prefers-reduced-motion: reduce) {
  .scan-line {
    animation: none;
  }
}
</style>
