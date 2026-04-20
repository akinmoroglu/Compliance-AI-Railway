<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import {
  IconCheck,
  IconAlertCircle,
  IconUpload,
  IconChevronDown,
  IconBrandMeta,
  IconBrandTiktok,
  IconBrandGoogle,
  IconLoader2,
  IconUsers
} from '@tabler/icons-vue'

const currentStep = ref(1)
const isChecking = ref(false)
const isSubmitting = ref(false)

const adFormat = ref<'single' | 'carousel'>('single')
const selectedPlatform = ref<'Meta' | ''>('')
const selectedRegion = ref('global')
const selectedMinAge = ref('25')
const selectedMaxAge = ref('34')

const adCopy = ref({ primaryText: '', headline: '', description: '' })
const landingPageUrl = ref('')
const urlError = ref('')
const uploadedFile = ref<string | null>(null)
const uploadedFileName = ref('')
const validationError = ref('')
const fileInputRef = ref<HTMLInputElement | null>(null)

interface AdCard { visual: string | null; headline: string; description: string }
const carouselCards = ref<AdCard[]>([{ visual: null, headline: '', description: '' }])
const activeCardIndex = ref(0)

const ageValue = (v: string) => v === '55+' ? 99 : parseInt(v)

const validMaxAgeOptions = computed(() => [
  { value: '24', label: '24' },
  { value: '34', label: '34' },
  { value: '44', label: '44' },
  { value: '55+', label: '55+' },
].filter(o => ageValue(o.value) > ageValue(selectedMinAge.value)))

watch(selectedMinAge, () => {
  if (ageValue(selectedMaxAge.value) <= ageValue(selectedMinAge.value)) {
    selectedMaxAge.value = validMaxAgeOptions.value[0]?.value ?? '55+'
  }
})

const parsedDomain = computed(() => {
  if (!landingPageUrl.value) return ''
  try {
    return new URL(landingPageUrl.value).hostname.replace(/^www\./, '').toUpperCase()
  } catch {
    return ''
  }
})

function validateUrl() {
  if (!landingPageUrl.value) { urlError.value = ''; return }
  try {
    new URL(landingPageUrl.value)
    urlError.value = ''
  } catch {
    urlError.value = 'Enter a valid URL (e.g. https://yoursite.com)'
  }
}

function handleFileUpload(e: Event) {
  const file = (e.target as HTMLInputElement).files?.[0]
  if (!file) return
  if (uploadedFile.value) URL.revokeObjectURL(uploadedFile.value)
  uploadedFile.value = URL.createObjectURL(file)
  uploadedFileName.value = file.name
  validationError.value = ''
}

function triggerUploadCard(index: number) {
  carouselCards.value[index].visual = `https://via.placeholder.com/300x300/1a1a2e/e94560?text=Card+${index + 1}`
}

function addCarouselCard() {
  if (carouselCards.value.length < 10) {
    carouselCards.value.push({ visual: null, headline: '', description: '' })
    activeCardIndex.value = carouselCards.value.length - 1
  }
}

function validateStep2(): boolean {
  if (urlError.value) {
    validationError.value = 'Please fix the landing page URL before continuing.'
    return false
  }
  if (adFormat.value === 'single' && !uploadedFile.value) {
    validationError.value = 'Please upload an image or video before checking compliance.'
    return false
  }
  if (adFormat.value === 'carousel' && !carouselCards.value[0].visual) {
    validationError.value = 'Please upload at least one carousel card image before checking compliance.'
    return false
  }
  validationError.value = ''
  return true
}

function checkCompliance() {
  if (!validateStep2()) return
  isSubmitting.value = true
  setTimeout(() => {
    isSubmitting.value = false
    isChecking.value = true
    currentStep.value = 3
    setTimeout(() => {
      isChecking.value = false
      currentStep.value = 4
    }, 2500)
  }, 400)
}

function nextStep() {
  currentStep.value++
}
</script>

<template>
  <div class="space-y-4">

    <!-- STEP 1: Platform Selection -->
    <div class="border border-border rounded-xl bg-card overflow-hidden">
      <button
        type="button"
        :aria-expanded="currentStep === 1"
        aria-controls="step1-panel"
        @click="currentStep = 1"
        class="w-full flex items-center justify-between p-4 sm:p-6 cursor-pointer hover:bg-muted/30 transition-colors select-none text-left"
      >
        <div class="flex items-center gap-3 sm:gap-4 min-w-0">
          <div class="h-8 w-8 shrink-0 rounded-lg bg-primary text-primary-foreground flex items-center justify-center font-semibold text-sm">
            1
          </div>
          <div class="min-w-0">
            <h2 class="text-base sm:text-lg font-semibold">Select Platform</h2>
            <p class="text-xs sm:text-sm text-muted-foreground hidden sm:block">Choose the platform you want to verify compliance for.</p>
          </div>
        </div>
        <IconChevronDown
          class="text-muted-foreground transition-transform duration-300 shrink-0 ml-2"
          :class="{ 'rotate-180': currentStep === 1 }"
        />
      </button>

      <div
        v-if="currentStep === 1"
        id="step1-panel"
        role="region"
        class="px-4 sm:px-6 pb-6 pt-2 border-t border-border animate-in fade-in slide-in-from-top-4"
      >
        <div class="grid grid-cols-1 sm:grid-cols-3 gap-4">
          <!-- Meta (Active) -->
          <label
            class="border-2 rounded-xl p-5 sm:p-6 transition-colors flex flex-col items-center justify-center text-center gap-3 relative cursor-pointer w-full focus-within:ring-4 focus-within:ring-primary/20"
            :class="selectedPlatform === 'Meta' ? 'border-primary bg-primary/5 hover:bg-primary/10 shadow-sm' : 'border-border bg-card hover:bg-muted/30 shadow-sm'"
          >
            <input type="radio" value="Meta" v-model="selectedPlatform" class="absolute opacity-0 w-0 h-0" />
            <div v-if="selectedPlatform === 'Meta'" class="absolute top-3 right-3 text-primary animate-in zoom-in">
              <IconCheck :size="20" stroke-width="3" />
            </div>
            <IconBrandMeta :size="48" :class="selectedPlatform === 'Meta' ? 'text-primary' : 'text-foreground'" />
            <div>
              <h3 class="font-bold text-foreground">Meta</h3>
              <p class="text-xs text-muted-foreground mt-1">Validates compliance within Meta's network (Facebook & Instagram).</p>
            </div>
          </label>

          <!-- Google (Coming Soon) -->
          <div
            class="border border-border rounded-xl p-5 sm:p-6 bg-muted/20 opacity-50 cursor-not-allowed flex flex-col items-center text-center gap-3 relative"
            title="Google Ads support coming in Q3 2026"
          >
            <IconBrandGoogle :size="48" class="text-muted-foreground" />
            <div>
              <h3 class="font-bold text-muted-foreground">Google Ads</h3>
              <p class="text-xs text-muted-foreground mt-1">Coming Soon — Q3 2026</p>
            </div>
          </div>

          <!-- TikTok (Coming Soon) -->
          <div
            class="border border-border rounded-xl p-5 sm:p-6 bg-muted/20 opacity-50 cursor-not-allowed flex flex-col items-center text-center gap-3 relative"
            title="TikTok support coming in Q3 2026"
          >
            <IconBrandTiktok :size="48" class="text-muted-foreground" />
            <div>
              <h3 class="font-bold text-muted-foreground">TikTok</h3>
              <p class="text-xs text-muted-foreground mt-1">Coming Soon — Q3 2026</p>
            </div>
          </div>
        </div>

        <!-- Target Demographics & Region -->
        <div class="mt-8 pt-6 border-t border-border grid grid-cols-1 lg:grid-cols-2 gap-4 pb-2">
          <div class="flex flex-col sm:flex-row sm:items-center gap-3 bg-muted/30 rounded-xl p-4 border border-border">
            <label for="region-select" class="font-bold text-sm text-foreground sm:whitespace-nowrap">Advertising Target Region</label>
            <div class="flex-1 relative">
              <select
                id="region-select"
                v-model="selectedRegion"
                class="w-full appearance-none bg-card border border-border rounded-lg px-4 py-2 pr-10 text-sm font-semibold focus:outline-none focus:ring-2 focus:ring-primary cursor-pointer text-foreground"
              >
                <option value="global">All Regions & Countries</option>
                <option value="us">United States</option>
                <option value="uk">United Kingdom</option>
                <option value="eu">European Union</option>
              </select>
              <IconChevronDown :size="16" class="absolute right-3 top-1/2 -translate-y-1/2 text-muted-foreground pointer-events-none" />
            </div>
          </div>

          <div class="flex flex-col sm:flex-row sm:items-center gap-3 bg-muted/30 rounded-xl p-4 border border-border">
            <div class="flex items-center gap-2 font-bold text-sm text-foreground sm:whitespace-nowrap">
              <IconUsers :size="20" class="text-muted-foreground shrink-0" />
              Age Range
            </div>
            <div class="flex items-center gap-2 sm:ml-2 flex-1">
              <div class="relative flex-1">
                <label for="min-age-select" class="sr-only">Minimum age</label>
                <select
                  id="min-age-select"
                  v-model="selectedMinAge"
                  class="w-full appearance-none bg-card border border-border rounded-lg px-3 py-2 pr-8 text-sm font-semibold focus:outline-none focus:ring-2 focus:ring-primary cursor-pointer text-foreground"
                >
                  <option value="18">18</option>
                  <option value="25">25</option>
                  <option value="35">35</option>
                </select>
                <IconChevronDown :size="16" class="absolute right-2 top-1/2 -translate-y-1/2 text-muted-foreground pointer-events-none" />
              </div>
              <span class="text-muted-foreground font-medium shrink-0">–</span>
              <div class="relative flex-1">
                <label for="max-age-select" class="sr-only">Maximum age</label>
                <select
                  id="max-age-select"
                  v-model="selectedMaxAge"
                  class="w-full appearance-none bg-card border border-border rounded-lg px-3 py-2 pr-8 text-sm font-semibold focus:outline-none focus:ring-2 focus:ring-primary cursor-pointer text-foreground"
                >
                  <option v-for="opt in validMaxAgeOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
                </select>
                <IconChevronDown :size="16" class="absolute right-2 top-1/2 -translate-y-1/2 text-muted-foreground pointer-events-none" />
              </div>
            </div>
          </div>
        </div>

        <div class="mt-6 flex justify-end">
          <button
            @click="nextStep"
            :disabled="!selectedPlatform"
            class="bg-primary text-primary-foreground px-6 py-2.5 rounded-lg font-medium hover:opacity-90 transition-opacity disabled:opacity-40 disabled:cursor-not-allowed"
          >
            Next Step
          </button>
        </div>
      </div>
    </div>

    <!-- STEP 2: Creative & Ad Copy -->
    <div class="border border-border rounded-xl bg-card overflow-hidden">
      <button
        type="button"
        :aria-expanded="currentStep === 2"
        aria-controls="step2-panel"
        @click="currentStep = 2"
        class="w-full flex items-center justify-between p-4 sm:p-6 cursor-pointer hover:bg-muted/30 transition-colors text-left"
      >
        <div class="flex items-center gap-3 sm:gap-4 min-w-0">
          <div
            class="h-8 w-8 shrink-0 rounded-lg flex items-center justify-center font-semibold text-sm transition-colors"
            :class="currentStep >= 2 ? 'bg-primary text-primary-foreground' : 'bg-muted text-muted-foreground'"
          >
            2
          </div>
          <div class="min-w-0">
            <h2 class="text-base sm:text-lg font-semibold" :class="currentStep < 2 && 'text-muted-foreground'">Ad Creative & Copy</h2>
            <p class="text-xs sm:text-sm text-muted-foreground hidden sm:block">Upload visual and provide the exact ad text for full compliance check.</p>
          </div>
        </div>
        <IconChevronDown
          class="text-muted-foreground transition-transform duration-300 shrink-0 ml-2"
          :class="{ 'rotate-180': currentStep === 2 }"
        />
      </button>

      <div
        v-if="currentStep === 2"
        id="step2-panel"
        role="region"
        class="px-4 sm:px-6 pb-6 pt-2 border-t border-border animate-in fade-in slide-in-from-top-4"
      >
        <!-- Ad Format Selector -->
        <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 mb-8 pb-4 border-b border-border">
          <div>
            <h3 class="font-bold text-lg text-foreground">Ad Format & Creative</h3>
            <p class="text-sm text-muted-foreground mt-1">Select your ad format and configure the visuals and copy.</p>
          </div>
          <div class="flex bg-muted/30 p-1 rounded-xl border border-border shrink-0">
            <button
              type="button"
              @click="adFormat = 'single'"
              :class="{'bg-card text-foreground shadow-sm ring-1 ring-border': adFormat === 'single', 'text-muted-foreground hover:text-foreground': adFormat !== 'single'}"
              class="px-4 py-2 rounded-lg text-sm font-bold transition-all"
            >
              Single Image or Video
            </button>
            <button
              type="button"
              @click="adFormat = 'carousel'"
              :class="{'bg-card text-foreground shadow-sm ring-1 ring-border': adFormat === 'carousel', 'text-muted-foreground hover:text-foreground': adFormat !== 'carousel'}"
              class="px-4 py-2 rounded-lg text-sm font-bold transition-all"
            >
              Carousel
            </button>
          </div>
        </div>

        <!-- SINGLE IMAGE FORMAT UI -->
        <div v-if="adFormat === 'single'" class="grid grid-cols-1 lg:grid-cols-2 gap-6 sm:gap-8">
          <div class="space-y-3">
            <h3 class="font-bold text-sm text-foreground">Visual Creative</h3>

            <!-- Hidden real file input -->
            <input
              ref="fileInputRef"
              type="file"
              accept="image/*,video/*"
              class="hidden"
              aria-label="Upload creative file"
              @change="handleFileUpload"
            />

            <div
              @click="fileInputRef?.click()"
              class="border-2 border-dashed border-border rounded-xl p-8 hover:bg-muted/50 transition-colors cursor-pointer flex flex-col items-center justify-center text-center group"
              :class="uploadedFile ? 'bg-muted/30 border-primary/50 min-h-[200px]' : 'min-h-[280px]'"
              role="button"
              tabindex="0"
              aria-label="Click to upload image or video"
              @keydown.enter="fileInputRef?.click()"
              @keydown.space.prevent="fileInputRef?.click()"
            >
              <template v-if="!uploadedFile">
                <div class="h-12 w-12 rounded-full bg-primary/10 flex items-center justify-center text-primary mb-4 group-hover:scale-110 transition-transform">
                  <IconUpload />
                </div>
                <p class="font-bold text-foreground">Upload your image or video</p>
                <p class="text-sm text-muted-foreground mt-1">Click here or drag & drop</p>
                <button
                  type="button"
                  @click.stop
                  disabled
                  class="mt-6 bg-secondary/50 text-secondary-foreground/60 px-5 py-2.5 rounded-lg text-sm font-bold cursor-not-allowed"
                  title="Image Library coming soon"
                >
                  Image Library
                </button>
              </template>
              <template v-else>
                <div class="w-full flex flex-col items-center justify-center space-y-4">
                  <div class="w-48 h-48 rounded-xl border-2 border-primary/30 shadow-inner overflow-hidden">
                    <img :src="uploadedFile" class="object-cover w-full h-full" :alt="uploadedFileName" />
                  </div>
                  <span class="text-xs text-muted-foreground">{{ uploadedFileName }}</span>
                  <span class="text-sm font-bold text-primary cursor-pointer hover:underline">Change File</span>
                </div>
              </template>
            </div>
          </div>

          <div class="space-y-4 bg-muted/20 p-5 sm:p-6 border border-border rounded-xl shadow-sm">
            <div class="border-b border-border pb-3 mb-2 flex items-center gap-2">
              <IconBrandMeta class="text-primary" />
              <h3 class="font-bold text-foreground">Meta Ad Copy</h3>
            </div>

            <div class="space-y-1.5">
              <label for="primary-text" class="text-xs font-bold text-muted-foreground uppercase tracking-wider">Primary Text</label>
              <textarea
                id="primary-text"
                v-model="adCopy.primaryText"
                rows="4"
                class="w-full rounded-lg border border-border bg-background px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-ring focus:border-transparent transition-shadow"
                placeholder="Appears above your ad..."
              ></textarea>
            </div>
            <div class="space-y-1.5">
              <label for="headline" class="text-xs font-bold text-muted-foreground uppercase tracking-wider">Headline</label>
              <input
                id="headline"
                v-model="adCopy.headline"
                type="text"
                class="w-full rounded-lg border border-border bg-background px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-ring focus:border-transparent transition-shadow"
                placeholder="Usually appears below the image..."
              />
            </div>
            <div class="space-y-1.5">
              <label for="description" class="text-xs font-bold text-muted-foreground uppercase tracking-wider">Description <span class="normal-case font-normal text-muted-foreground/70">(Optional)</span></label>
              <input
                id="description"
                v-model="adCopy.description"
                type="text"
                class="w-full rounded-lg border border-border bg-background px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-ring focus:border-transparent transition-shadow"
                placeholder="Additional context..."
              />
            </div>
            <div class="space-y-1.5 pt-2 border-t border-border mt-2">
              <label for="landing-url" class="text-xs font-bold text-muted-foreground uppercase tracking-wider">
                Landing Page URL <span class="normal-case font-normal text-muted-foreground/70">(Recommended)</span>
              </label>
              <input
                id="landing-url"
                v-model="landingPageUrl"
                type="url"
                class="w-full rounded-lg border border-border bg-background px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-ring focus:border-transparent transition-shadow"
                :class="urlError ? 'border-destructive focus:ring-destructive' : ''"
                placeholder="https://yoursite.com/product-page"
                @blur="validateUrl"
              />
              <p v-if="urlError" class="text-xs text-destructive font-medium">{{ urlError }}</p>
              <p v-else class="text-[11px] text-muted-foreground">Meta checks your landing page too. Skipping this may miss destination-level violations.</p>
            </div>
          </div>
        </div>

        <!-- CAROUSEL FORMAT UI -->
        <div v-else class="space-y-8">
          <div class="grid grid-cols-1 lg:grid-cols-2 gap-4">
            <div class="space-y-1.5">
              <label for="carousel-primary-text" class="text-xs font-bold text-muted-foreground uppercase tracking-wider flex items-center gap-2">
                <IconBrandMeta class="text-primary w-4 h-4" /> Global Primary Text
              </label>
              <textarea
                id="carousel-primary-text"
                v-model="adCopy.primaryText"
                rows="2"
                class="w-full rounded-lg border border-border bg-background px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-ring focus:border-transparent transition-shadow"
                placeholder="This text appears above the entire carousel..."
              ></textarea>
            </div>
            <div class="space-y-1.5">
              <label for="carousel-landing-url" class="text-xs font-bold text-muted-foreground uppercase tracking-wider">
                Landing Page URL <span class="normal-case font-normal text-muted-foreground/70">(Recommended)</span>
              </label>
              <input
                id="carousel-landing-url"
                v-model="landingPageUrl"
                type="url"
                class="w-full rounded-lg border border-border bg-background px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-ring focus:border-transparent transition-shadow"
                :class="urlError ? 'border-destructive focus:ring-destructive' : ''"
                placeholder="https://yoursite.com/product-page"
                @blur="validateUrl"
              />
              <p v-if="urlError" class="text-xs text-destructive font-medium">{{ urlError }}</p>
              <p v-else class="text-[11px] text-muted-foreground">Meta checks your landing page too. Skipping this may miss destination-level violations.</p>
            </div>
          </div>

          <div class="grid grid-cols-1 lg:grid-cols-3 gap-6 sm:gap-8">
            <div class="lg:col-span-2 border border-border rounded-xl bg-muted/10 p-5 overflow-hidden flex flex-col">
              <h3 class="font-bold text-sm text-foreground mb-4">Carousel Cards ({{ carouselCards.length }}/10)</h3>
              <div class="flex gap-4 overflow-x-auto pb-4 flex-1 items-center">
                <div
                  v-for="(card, idx) in carouselCards"
                  :key="idx"
                  @click="activeCardIndex = idx"
                  class="shrink-0 w-44 h-60 rounded-xl border-2 transition-all cursor-pointer flex flex-col items-center justify-center relative bg-card overflow-hidden group"
                  :class="activeCardIndex === idx ? 'border-primary shadow-md scale-105' : 'border-dashed border-border hover:border-solid hover:border-primary/50'"
                >
                  <div v-if="activeCardIndex === idx" class="absolute top-2 right-2 bg-primary text-primary-foreground text-[10px] font-bold px-2 py-0.5 rounded-full z-10">Editing</div>
                  <template v-if="!card.visual">
                    <IconUpload class="text-muted-foreground group-hover:text-primary transition-colors mb-2" />
                    <span class="text-xs font-bold text-muted-foreground">Click to upload</span>
                    <span class="text-[10px] text-muted-foreground/70 mt-1">Card {{ idx + 1 }}</span>
                  </template>
                  <template v-else>
                    <img :src="card.visual" class="w-full h-full object-cover opacity-90" :alt="`Card ${idx + 1}`" />
                    <div class="absolute bottom-0 w-full bg-background/90 backdrop-blur border-t border-border p-2">
                      <div class="text-[10px] font-bold truncate">{{ card.headline || 'Headline' }}</div>
                    </div>
                  </template>
                </div>

                <div
                  v-if="carouselCards.length < 10"
                  @click="addCarouselCard"
                  class="shrink-0 w-16 h-60 border-2 border-dashed border-border rounded-xl hover:bg-muted/50 transition-colors cursor-pointer flex items-center justify-center text-muted-foreground hover:text-foreground"
                  role="button"
                  aria-label="Add carousel card"
                >
                  <span class="text-2xl font-light">+</span>
                </div>
              </div>
            </div>

            <div class="space-y-4 bg-muted/20 p-5 border border-border rounded-xl shadow-sm h-fit">
              <div class="border-b border-border pb-3 mb-2 flex items-center justify-between">
                <h3 class="font-bold text-foreground text-sm">Editing Card {{ activeCardIndex + 1 }}</h3>
              </div>
              <div class="space-y-1.5">
                <label :for="`card-media-${activeCardIndex}`" class="text-xs font-bold text-muted-foreground uppercase tracking-wider">Card Media</label>
                <div
                  :id="`card-media-${activeCardIndex}`"
                  @click="triggerUploadCard(activeCardIndex)"
                  class="w-full h-24 border border-dashed border-border rounded-lg bg-card hover:bg-muted/50 cursor-pointer flex items-center justify-center transition-colors"
                  role="button"
                  :aria-label="`Upload media for card ${activeCardIndex + 1}`"
                >
                  <span v-if="!carouselCards[activeCardIndex].visual" class="text-xs font-bold text-muted-foreground flex items-center gap-2"><IconUpload :size="14" /> Upload Media</span>
                  <span v-else class="text-xs font-bold text-primary flex items-center gap-2">Replace Image</span>
                </div>
              </div>
              <div class="space-y-1.5">
                <label :for="`card-headline-${activeCardIndex}`" class="text-xs font-bold text-muted-foreground uppercase tracking-wider">Card Headline</label>
                <input
                  :id="`card-headline-${activeCardIndex}`"
                  v-model="carouselCards[activeCardIndex].headline"
                  type="text"
                  class="w-full rounded-lg border border-border bg-background px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-ring focus:border-transparent transition-shadow"
                  placeholder="Short headline..."
                />
              </div>
              <div class="space-y-1.5">
                <label :for="`card-description-${activeCardIndex}`" class="text-xs font-bold text-muted-foreground uppercase tracking-wider">Description</label>
                <input
                  :id="`card-description-${activeCardIndex}`"
                  v-model="carouselCards[activeCardIndex].description"
                  type="text"
                  class="w-full rounded-lg border border-border bg-background px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-ring focus:border-transparent transition-shadow"
                  placeholder="Sub-text..."
                />
              </div>
            </div>
          </div>
        </div>

        <!-- Validation error + Submit -->
        <div class="mt-8 flex flex-col sm:flex-row sm:items-center justify-end gap-3 pt-6 border-t border-border">
          <p v-if="validationError" role="alert" class="text-sm text-destructive font-medium flex items-center gap-1.5">
            <IconAlertCircle :size="16" class="shrink-0" />
            {{ validationError }}
          </p>
          <button
            type="button"
            @click="checkCompliance"
            :disabled="isSubmitting"
            class="bg-[#ed3875] text-white px-8 py-3 rounded-lg font-bold hover:opacity-90 transition-opacity disabled:opacity-70 disabled:cursor-not-allowed shadow-md flex items-center gap-2 shrink-0"
          >
            <IconLoader2 v-if="isSubmitting" :size="16" class="animate-spin" />
            <span>{{ isSubmitting ? 'Checking…' : 'Check Compliance >' }}</span>
          </button>
        </div>
      </div>
    </div>

    <!-- STEP 3: Processing State -->
    <div v-if="currentStep === 3" class="border border-border rounded-xl bg-card p-10 sm:p-12 flex flex-col items-center justify-center text-center animate-in fade-in zoom-in-95">
      <IconLoader2 class="animate-spin text-primary mb-6" :size="48" />
      <h2 class="text-xl font-bold text-primary mb-2">AI is analyzing your ad…</h2>
      <p class="text-muted-foreground">Checking Meta policies for health claims, sensationalism, and visual compliance.</p>
      <div class="mt-8 space-y-3 w-full max-w-md">
        <div class="bg-primary/5 border border-primary/20 rounded-lg p-3 flex items-center justify-between">
          <span class="text-sm font-medium">Analyzing wording & copy</span>
          <IconCheck class="text-primary" :size="16" />
        </div>
        <div class="bg-muted/50 border border-border rounded-lg p-3 flex items-center justify-between opacity-70">
          <span class="text-sm font-medium">Scanning visual elements</span>
          <IconLoader2 class="text-muted-foreground animate-spin" :size="16" />
        </div>
      </div>
    </div>

    <!-- STEP 4: Results Dashboard -->
    <div v-if="currentStep === 4" class="animate-in fade-in slide-in-from-bottom-4">
      <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-3 mb-6">
        <div>
          <h2 class="text-xl sm:text-2xl font-bold text-foreground">Compliance Report</h2>
          <p class="text-muted-foreground mt-1">Review the issues found within your ad creative and copy.</p>
        </div>
        <button @click="currentStep = 1" class="text-sm font-medium text-primary hover:underline shrink-0">Start New Check</button>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-5 gap-6">

        <!-- Left: Ad Preview (2 cols) -->
        <div class="lg:col-span-2 space-y-4">
          <div class="border border-border rounded-xl bg-card p-4">
            <!-- Simulated Facebook Ad Card -->
            <div class="flex items-center gap-2 mb-3">
              <div class="w-8 h-8 rounded-full bg-muted shrink-0"></div>
              <div>
                <div class="text-xs font-bold text-foreground">Your Brand</div>
                <div class="text-[10px] text-muted-foreground">Sponsored</div>
              </div>
            </div>

            <p v-if="adCopy.primaryText" class="text-sm text-foreground mb-3">{{ adCopy.primaryText }}</p>

            <!-- Image preview: constrained height, shows actual upload or placeholder frame -->
            <div class="w-full max-h-64 aspect-video bg-muted/30 rounded overflow-hidden flex items-center justify-center shadow-inner border border-border">
              <img v-if="uploadedFile" :src="uploadedFile" class="w-full h-full object-cover" :alt="uploadedFileName" />
              <span v-else class="text-xs text-muted-foreground font-medium">No image uploaded</span>
            </div>

            <div class="bg-muted/30 p-3 mt-2 border border-border flex justify-between items-center gap-2">
              <div class="min-w-0">
                <p v-if="parsedDomain" class="text-xs text-muted-foreground uppercase truncate">{{ parsedDomain }}</p>
                <p v-if="adCopy.headline" class="text-sm font-bold truncate">{{ adCopy.headline }}</p>
                <p v-if="adCopy.description" class="text-xs text-muted-foreground truncate">{{ adCopy.description }}</p>
              </div>
              <button type="button" class="bg-secondary text-secondary-foreground text-xs font-bold px-3 py-1 rounded shrink-0">Learn More</button>
            </div>
          </div>
        </div>

        <!-- Right: Results Checklist (3 cols) -->
        <div class="lg:col-span-3 space-y-4">
          <div class="border border-border rounded-xl bg-card overflow-hidden">
            <div class="bg-destructive/10 border-b border-destructive/20 p-4 flex items-center justify-between">
              <div class="flex items-center gap-2 text-destructive font-bold">
                <IconAlertCircle :size="20" />
                Meta Compliance Issues
              </div>
              <div class="bg-destructive text-destructive-foreground text-xs font-bold px-2 py-0.5 rounded-full">Failed: High Risk</div>
            </div>

            <div class="p-5 sm:p-6 space-y-6">
              <div>
                <div class="flex items-start gap-2">
                  <IconAlertCircle class="text-destructive mt-0.5 shrink-0" :size="16" />
                  <div>
                    <h4 class="font-bold text-sm text-foreground">Misleading Health Claims (Copy)</h4>
                    <p class="text-sm text-muted-foreground mt-1">
                      The primary text contains extreme weight loss claims with a guaranteed result. Meta severely restricts promises of guaranteed physical results without robust scientific backings or disclaimers.
                    </p>
                  </div>
                </div>
              </div>
              <div>
                <div class="flex items-start gap-2">
                  <IconAlertCircle class="text-destructive mt-0.5 shrink-0" :size="16" />
                  <div>
                    <h4 class="font-bold text-sm text-foreground">Unrealistic Body Standards (Visual)</h4>
                    <p class="text-sm text-muted-foreground mt-1">
                      The uploaded image may show a before-and-after scenario. Meta prohibits before-and-after images that promote sudden weight loss or invoke negative self-perception.
                    </p>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="bg-muted p-4 rounded-xl border border-border flex items-start gap-3">
            <IconCheck class="text-muted-foreground shrink-0 mt-0.5" />
            <p class="text-sm text-muted-foreground">No trademark infringements or prohibited profanity detected in the ad copy or visual text overlay.</p>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>
