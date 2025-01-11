<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useRoute } from 'vue-router';

definePageMeta({
  layout: 'empty',
  title: 'Verifying Email',
  preview: {
    title: 'Verify Email',
    description: 'For account verification',
    categories: ['layouts', 'authentication'],
    src: '/img/screens/auth-recover.png',
    srcDark: '/img/screens/auth-recover-dark.png',
    order: 103,
  },
});

const success = ref(false);
const message = ref('  Please wait while we verify your account...');
const loginButtonVisible = ref(false);
const route = useRoute();

onMounted(async () => {
  const token = route.query.token; // Get token from query parameter
  if (token) {
    try {
      const response = await fetch('http://localhost:3005/verifyuseremail', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ token }),
      });

      const data = await response.json();
      if (response.ok) {
        message.value = 'Congratulations! Your account is now verified. You may now log in!';
        success.value = true;
        loginButtonVisible.value = true;
      } else {
        message.value = data.message || 'Verification failed. Please try again.';
      }
    } catch (error) {
      message.value = 'An error occurred. Please try again.';
      console.error(error);
    }
  } else {
    message.value = 'Invalid verification link.';
  }
});
</script>

<template>
  <div
    class="bg-muted-100 dark:bg-muted-900 relative min-h-screen w-full overflow-hidden px-4"
  >
    <div
      class="mx-auto flex h-16 w-full max-w-6xl items-center justify-between px-4"
    >
      <NuxtLink
        to="/dashboards"
        class="text-muted-400 hover:text-primary-500 dark:text-muted-700 dark:hover:text-primary-500 transition-colors duration-300"
      >
        <TairoLogo class="h-10 w-10" />
      </NuxtLink>
      <div>
        <BaseThemeToggle />
      </div>
    </div>
    <div class="flex w-full items-center justify-center">
      <div class="relative mx-auto w-full max-w-2xl">
        <!--Form-->
        <div class="me-auto ms-auto mt-4 w-full">
          <div class="me-auto ms-auto mt-4 w-full max-w-md">
            <div class="text-center"></div>
            <Transition
              mode="out-in"
              enter-active-class="transition-all duration-300 ease-out"
              enter-from-class="scale-0 opacity-0"
              enter-to-class="scale-100 opacity-100"
              leave-active-class="transition-all duration-75 ease-in"
              leave-from-class="scale-100 opacity-100"
              leave-to-class="scale-0 opacity-0"
            >
              <div v-if="success" key="success" class="px-8 py-4">
                <div class="mb-4 space-y-4">
                  <BaseHeading as="h2" size="3xl" weight="medium">
                    Email successfully verified!
                  </BaseHeading>

                  <BaseMessage class="p-6" :closable="false">
                    <p class="text-base">
                      {{ message }}
                    </p>
                  </BaseMessage>

                  <div v-if="loginButtonVisible" class="mt-4">
                    <NuxtLink
                      to="/login"
                      class="bg-primary-500 hover:bg-primary-700 text-white px-4 py-2 rounded-md transition"
                    >
                      Log In
                    </NuxtLink>
                  </div>
                </div>
              </div>
              <div v-else key="verifying" class="px-8 py-4">
                <div class="mb-4 space-y-4">
                  <BaseHeading as="h2" size="3xl" weight="medium">
                    Verifying your account...
                  </BaseHeading>
                  <BaseMessage class="p-6" :closable="false">
                    <p class="text-base">{{ message }}</p>
                  </BaseMessage>
                </div>
              </div>
            </Transition>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
