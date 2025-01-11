<script setup lang="ts">
import { ref, computed } from 'vue';
import { Field, useForm } from 'vee-validate';
import { useRouter } from 'vue-router';


const value = ref()

// const Gender = ['Male', 'Female', 'Others']

// Gender options
const genderOptions = [ 'Male','Female','Other'];






definePageMeta({
  layout: 'empty',
  title: 'Complete Profile',
  preview: {
    title: 'Profile',
    description: 'For authentication and sign in',
    categories: ['layouts', 'authentication'],
    src: '/img/screens/auth-login-3.png',
    srcDark: '/img/screens/auth-login-3-dark.png',
    order: 98,
  },
})

interface ProfileResponse {
  successfull: boolean;
  message: string;
}

// Define state variables
const email = ref('');
const firstName = ref('');
const lastName = ref('');
const phone = ref('');
const address = ref('');
const gender = ref('');
const errorMsg = ref('');

// Router instance
const router = useRouter();

// Handle profile submission
const handleProfile = async () => {
  email.value = localStorage.getItem('email') || '';

  try {
    const response = await fetch('http://localhost:3004/profile', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        email: email.value,
        firstName: firstName.value,
        lastName: lastName.value,
        phone: phone.value,
        address: address.value,
        gender: gender.value,
      }),
    });

    const res: ProfileResponse = await response.json();
    if (res.successfull) {
      router.push('/user');
    } else {
      errorMsg.value = res.message;
    }
  } catch (error: any) {
    errorMsg.value = error.message;
    console.error('Error submitting profile:', error);
  }
};
</script>

<template>

  <div class="bg-muted-100 dark:bg-muted-900 relative min-h-screen w-full overflow-hidden px-4" >

    <div  class="mx-auto flex h-16 w-full max-w-6xl items-center justify-between px-4" >

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
        <div class="me-auto ms-auto mt-4 w-full">
          <form
            @submit.prevent="handleProfile"
            class="me-auto ms-auto mt-4 w-full max-w-md"
            novalidate
          >
            <div class="text-center">
              <BaseHeading as="h2" size="3xl" weight="medium">
                Welcome, Kindly Complete Your Registration
              </BaseHeading>
              <BaseParagraph size="sm" class="text-muted-400 mb-6">
                Let's start with your personal details.
              </BaseParagraph>
            </div>
            <div class="px-8 py-4">
              <div class="mb-4 space-y-4">
                <!-- First Name -->
                <Field name="firstname">
                  <BaseInput
                    v-model="firstName"
                    type="text"
                    label="First Name"
                    placeholder="e.g., Gideon"
                    autocomplete="given-name"
                    :classes="{ input: 'h-12' }"
                  />
                </Field>

                <!-- Last Name -->
                <Field name="lastname">
                  <BaseInput
                    v-model="lastName"
                    type="text"
                    label="Last Name"
                    placeholder="e.g., Akpanoko"
                    autocomplete="family-name"
                    :classes="{ input: 'h-12' }"
                  />
                </Field>

                <!-- Gender -->
                <Field name="gender">
                    <div>
                     

                      <!-- Gender Selection using BaseListbox -->
                      <BaseListbox
                        v-model="gender"
                        :items="genderOptions"
                        label="gender"
                        placeholder="Select Gender"
                        size="md"
                        rounded="sm"
                        :classes="{ button: 'h-12 w-full' }"
                      />

                      <p v-if="!gender" class="text-danger-500 text-sm mt-1">
                        Please select your gender.
                      </p>
                    </div>
                  </Field>



                <!-- <Field name="gender">
                  <div>
                    <BaseHeading as="h4" size="sm" weight="medium" class="mb-2">
                      Gender
                    </BaseHeading>


                    <BaseListbox
                      v-model="value"
                      label="Gender Tick"
                      :items="Gender"
                      placeholder="Select a Gender"
                      rounded="none"
                    />

                    <BaseDropdown
                      variant="button"
                      label="Select Gender"
                      placement="bottom-start"
                      size="md"
                      :classes="{ button: 'h-12 w-full' }"
                    >
                      <BaseDropdownItem
                        @click="gender = 'Male'"
                        title="Male"
                        text="Male"
                        rounded="sm"
                      />
                      <BaseDropdownItem
                        @click="gender = 'Female'"
                        title="Female"
                        text="Female"
                        rounded="sm"
                      />
                      <BaseDropdownItem
                        @click="gender = 'other'"
                        title="Other"
                        text="Other"
                        rounded="sm"
                      />
                    </BaseDropdown>
                    <p v-if="!gender" class="text-danger-500 text-sm mt-1">
                      Please select your gender.
                    </p>
                  </div>
                </Field> -->

                <!-- Phone Number -->
                <Field name="phonenumber">
                  <BaseInput
                    v-model="phone"
                    type="tel"
                    label="Phone Number"
                    placeholder="e.g., 08097180482"
                    autocomplete="tel"
                    :classes="{ input: 'h-12' }"
                  />
                </Field>

                <!-- Address -->
                <Field name="address">
                  <BaseInput
                    v-model="address"
                    type="text"
                    label="Home Address"
                    placeholder="e.g., 123 Main Street"
                    autocomplete="street-address"
                    :classes="{ input: 'h-12' }"
                  />
                </Field>
              </div>

              <!-- Submit Button -->
              <div class="mb-6">
                <BaseButton
                  :disabled="!firstName || !lastName || !phone || !address || !gender"
                  :loading="false"
                  type="submit"
                  color="primary"
                  class="!h-12 w-full"
                >
                  Complete Profile
                </BaseButton>
                <div> 
                  <p class="text-muted-400 mt-4 flex justify-between font-sans text-sm leading-5">
                    <span> </span>
                  <NuxtLink
                       to="/login"
                       class="text-primary-600 hover:text-primary-500 font-medium underline-offset-4 transition duration-150 ease-in-out hover:underline"
                >
                  Sign Out
                </NuxtLink>
              </p> 
            
            </div>
                <p v-if="errorMsg" class="text-danger-500 mt-2">{{ errorMsg }}</p>
              </div>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>


</template>
