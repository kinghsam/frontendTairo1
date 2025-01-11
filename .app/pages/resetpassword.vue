<script setup lang="ts">
import { toTypedSchema } from '@vee-validate/zod';
import { Field, useForm } from 'vee-validate';
import { z } from 'zod';
import { ref, computed } from 'vue';
import { useRoute } from 'vue-router';

definePageMeta({
  layout: 'empty',
  title: 'Reset Password',
  preview: {
    title: 'Reset Password',
    description: 'For password reset',
    categories: ['layouts', 'authentication'],
    src: '/img/screens/auth-recover.png',
    srcDark: '/img/screens/auth-recover-dark.png',
    order: 103,
  },
});

const route = useRoute();

const VALIDATION_TEXT = {
  PASSWORD_REQUIRED: 'Password is required',
  CONFIRM_PASSWORD_REQUIRED: 'Confirm password is required',
  PASSWORDS_DO_NOT_MATCH: 'Passwords do not match',
};

// Define Zod schema for form validation
const zodSchema = z.object({
  password: z.string().min(8, VALIDATION_TEXT.PASSWORD_REQUIRED),
  confirmPassword: z.string().min(8, VALIDATION_TEXT.CONFIRM_PASSWORD_REQUIRED),
}).refine((data) => data.password === data.confirmPassword, {
  message: VALIDATION_TEXT.PASSWORDS_DO_NOT_MATCH,
  path: ['confirmPassword'], // Path to show error for mismatched passwords
});

// Infer TypeScript type from Zod schema
type FormInput = z.infer<typeof zodSchema>;

const validationSchema = toTypedSchema(zodSchema);
const initialValues = computed<FormInput>(() => ({
  password: '',
  confirmPassword: '',
}));

const { handleSubmit, isSubmitting, errors } = useForm({
  validationSchema,
  initialValues,
});

const Message = ref('');
const errorMessage = ref('');

// Handle form submission
const onSubmit = handleSubmit(async (values) => {
  Message.value = '';
  errorMessage.value = '';

  const token = route.query.token as string;
  const email = route.query.email as string;

  if (!token || !email) {
    errorMessage.value = 'Invalid request. Missing token or email.';
    return;
  }

  try {
    const response = await fetch('http://localhost:3005/forgetpassword/reset', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        email,
        token,
        newPassword: values.password,
      }),
    });

    if (!response.ok) {
      throw new Error('Failed to reset password. Please try again.');
    }

    const data = await response.json();
    Message.value = data.message || 'Password reset successfully!';
  } catch (error) {
    console.error('Error during API call:', error);
    errorMessage.value = 'An error occurred. Please try again later.';
  }
});
</script>

<template>
  <div class="bg-muted-100 dark:bg-muted-900 relative min-h-screen w-full overflow-hidden px-4">
    <div class="mx-auto flex h-16 w-full max-w-6xl items-center justify-between px-4">
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
        <div class="me-auto ms-auto mt-4 w-full max-w-md">
          <div class="text-center">
            <BaseHeading as="h2" size="3xl" weight="medium">
              Reset Your Password
            </BaseHeading>
            <BaseParagraph size="sm" class="text-muted-400 mb-6">
              Follow the instructions to reset your password.
            </BaseParagraph>
          </div>

          <Transition
            mode="out-in"
            enter-active-class="transition-all duration-300 ease-out"
            enter-from-class="scale-0 opacity-0"
            enter-to-class="scale-100 opacity-100"
            leave-active-class="transition-all duration-75 ease-in"
            leave-from-class="scale-100 opacity-100"
            leave-to-class="scale-0 opacity-0"
          >
            <div v-if="Message" class="px-8 py-4">
              <BaseMessage class="p-6" :closable="false">
                <p class="text-base">{{ Message }}</p>
              </BaseMessage>
            </div>

            <form
              v-else
              @submit.prevent="onSubmit"
              class="px-8 py-4"
              novalidate
            >
              <div class="mb-4 space-y-4">
                <!-- Password Field -->
                <Field
                  v-slot="{ field, errorMessage, handleChange, handleBlur }"
                  name="password"
                >
                  <BaseInput
                    :model-value="field.value"
                    :error="errorMessage"
                    :disabled="isSubmitting"
                    type="password"
                    label="Password"
                    placeholder="••••••••••"
                    autocomplete="new-password"
                    :classes="{ input: 'h-12' }"
                    @update:model-value="handleChange"
                    @blur="handleBlur"
                  />
                </Field>

                <!-- Confirm Password Field -->
                <Field
                  v-slot="{ field, errorMessage, handleChange, handleBlur }"
                  name="confirmPassword"
                >
                  <BaseInput
                    :model-value="field.value"
                    :error="errorMessage"
                    :disabled="isSubmitting"
                    type="password"
                    label="Confirm Password"
                    placeholder="••••••••••"
                    :classes="{ input: 'h-12' }"
                    @update:model-value="handleChange"
                    @blur="handleBlur"
                  />
                </Field>
              </div>

              <div class="mb-6">
                <BaseButton
                  :disabled="isSubmitting"
                  :loading="isSubmitting"
                  type="submit"
                  color="primary"
                  class="!h-12 w-full"
                >
                  Reset Password
                </BaseButton>
              </div>
            </form>
          </Transition>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
</style>
