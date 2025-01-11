<script setup lang="ts">
import { ref } from 'vue'
import { toTypedSchema } from '@vee-validate/zod'
import { Field, useForm } from 'vee-validate'
import { z } from 'zod'

definePageMeta({
  layout: 'empty',
  title: 'Signup',
})

const VALIDATION_TEXT = {
  EMAIL_REQUIRED: 'A valid email is required',
  PASSWORD_LENGTH: 'Password must be at least 8 characters',
  PASSWORD_CONTAINS_EMAIL: 'Password cannot contain your email',
  PASSWORD_MATCH: 'Passwords do not match',
  TERMS_REQUIRED: 'You must agree to the terms and conditions',
}

const zodSchema = z
  .object({
    username: z.string().nonempty('Username is required'),
    email: z.string().email(VALIDATION_TEXT.EMAIL_REQUIRED),
    password: z.string().min(8, VALIDATION_TEXT.PASSWORD_LENGTH),
    confirmPassword: z.string(),
    terms: z.boolean(),
  })
  .superRefine((data, ctx) => {
    if (data.password.includes(data.email)) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: VALIDATION_TEXT.PASSWORD_CONTAINS_EMAIL,
        path: ['password'],
      })
    }
    if (data.password !== data.confirmPassword) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: VALIDATION_TEXT.PASSWORD_MATCH,
        path: ['confirmPassword'],
      })
    }
    if (!data.terms) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: VALIDATION_TEXT.TERMS_REQUIRED,
        path: ['terms'],
      })
    }
  })

type FormInput = z.infer<typeof zodSchema>

const validationSchema = toTypedSchema(zodSchema)
const initialValues = computed<FormInput>(() => ({
  username: '',
  email: '',
  password: '',
  confirmPassword: '',
  terms: false,
}))

const { handleSubmit, isSubmitting } = useForm({
  validationSchema,
  initialValues,
})

const toaster = useToaster()

const onSubmit = handleSubmit(async (values) => {
  try {
    const response = await fetch('http://localhost:3004/confirmemail', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        username: values.username,
        email: values.email,
        password: values.password,
      }),
    })

    const data = await response.json()

    if (response.ok) {
      toaster.show({
        title: 'Success',
        message: 'Signup successful! Please check your email to confirm.',
        color: 'success',
      })
    } else {
      toaster.show({
        title: 'Error',
        message: data.message || 'Signup failed. Please try again.',
        color: 'danger',
      })
    }
  } catch (error) {
    console.error('Signup error:', error)
    toaster.show({
      title: 'Error',
      message: 'Something went wrong. Please try again.',
      color: 'danger',
    })
  }
})
</script>



<template>
  <div
    class="bg-muted-100 dark:bg-muted-900 relative min-h-screen w-full overflow-hidden px-4"
  >
    <div
      class="mx-auto flex h-16 w-full max-w-6xl items-center justify-between px-4"
    >
      <NuxtLink
        to="/"
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
        <!-- Form -->
        <div class="me-auto ms-auto mt-4 w-full">
          <form
            method="POST"
            action=""
            @submit.prevent="onSubmit"
            class="me-auto ms-auto mt-4 w-full max-w-md"
            novalidate
          >
            <div class="text-center">
              <BaseHeading as="h2" size="3xl" weight="medium">
                Welcome
              </BaseHeading>
              <BaseParagraph size="sm" class="text-muted-400 mb-6">
                Let's start by creating your account
              </BaseParagraph>
            </div>
            <div class="px-8 py-4">
              <div class="mb-4 space-y-4">
                <!-- Username Field -->
                <Field
                  v-slot="{ field, errorMessage, handleChange, handleBlur }"
                  name="username"
                >
                  <BaseInput
                    :model-value="field.value"
                    :error="errorMessage"
                    :disabled="isSubmitting"
                    type="text"
                    label="Username"
                    placeholder="ex: user001"
                    autocomplete="username"
                    :classes="{ input: 'h-12' }"
                    @update:model-value="handleChange"
                    @blur="handleBlur"
                  />
                </Field>

                <!-- Email Field -->
                <Field
                  v-slot="{ field, errorMessage, handleChange, handleBlur }"
                  name="email"
                >
                  <BaseInput
                    :model-value="field.value"
                    :error="errorMessage"
                    :disabled="isSubmitting"
                    type="email"
                    label="Email address"
                    placeholder="ex: maya@example.com"
                    autocomplete="email"
                    :classes="{ input: 'h-12' }"
                    @update:model-value="handleChange"
                    @blur="handleBlur"
                  />
                </Field>

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

              <!-- Terms Checkbox -->
              <div class="mb-6">
                <div class="mt-6 flex items-center justify-between">
                  <Field
                    v-slot="{ field, errorMessage, handleChange, handleBlur }"
                    name="terms"
                  >
                    <BaseCheckbox
                      :model-value="field.value"
                      :disabled="isSubmitting"
                      :error="errorMessage"
                      shape="rounded"
                      color="primary"
                      @update:model-value="handleChange"
                      @blur="handleBlur"
                    >
                      <span>
                        <span>I accept the</span>
                        <a
                          href="#"
                          class="text-primary-500 hover:underline focus:underline"
                        >
                          Terms of Service
                        </a>
                      </span>
                    </BaseCheckbox>
                  </Field>
                </div>
              </div>

              <!-- Submit Button -->
              <div class="mb-6">
                <BaseButton
                  :disabled="isSubmitting"
                  :loading="isSubmitting"
                  type="submit"
                  color="primary"
                  class="!h-12 w-full"
                >
                  Sign Up
                </BaseButton>
              </div>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>
