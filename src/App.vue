<script setup lang="ts">
import { IconShieldCheck } from '@tabler/icons-vue'
import ComplianceDashboard from './components/ComplianceDashboard.vue'
</script>

<template>
  <div class="min-h-screen bg-background text-foreground flex flex-col">
    <!-- Header -->
    <header class="border-b border-border bg-card px-4 sm:px-8 py-4 flex items-center shrink-0 min-h-[64px]">
      <div class="flex items-center gap-3 min-w-0">
        <div class="h-9 w-9 shrink-0 flex items-center justify-center rounded-full bg-primary/10 text-primary">
          <IconShieldCheck :size="22" stroke-width="2" />
        </div>
        <div class="min-w-0">
          <h1 class="text-base sm:text-xl font-bold text-primary truncate">Compliance Checker AI</h1>
          <p class="text-xs sm:text-sm text-muted-foreground hidden sm:block">Verify ad compliance for platform, brand, and legal standards.</p>
        </div>
      </div>
    </header>

    <!-- Main Content -->
    <main class="flex-1 p-4 sm:p-8 max-w-5xl mx-auto w-full">
      <ComplianceDashboard />
    </main>
  </div>
</template>
