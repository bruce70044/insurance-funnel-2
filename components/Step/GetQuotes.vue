<template lang="html">
  <ValidationObserver ref="observer" class="w-full mt-6" tag="div">
    <Title>{{ $store.state.name.first }}, Last Step!</Title>
    <div>
      <div class="flex flex-col">
        <label for="address" class="text-xl mb-2">Street Address <span class="text-red-500">*</span></label>
        <TextInput
          v-model="streetAddress"
          id="address"
          rules="street-address"
          :errors="[errors.address]" />
        <div class="font-thin text-indigo-600 text-sm mb-6">
          {{ city }}, {{ state }} {{ $store.state.zipcode }}
        </div>
      </div>

      <div class="flex flex-col">
        <label for="phone_number" class="text-xl mb-2">Phone Number <span class="text-red-500">*</span></label>
        <TextInput
          v-model="phoneNumber"
          id="phone_number"
          rules="phone-number"
          :errors="[errors.phoneNumber]" />
      </div>
    </div>
    <PrimaryButton
      class="mmd:w-full"
      :bold="true"
      :disabled="submitted"
      @click="submit"
      >Get My Auto Quotes</PrimaryButton
    >
    <div class="mt-16 mb-10 space-y-4 text-xs text-gray-400 sm:mt-12">
      <div>
        <span
          >By clicking "Get My Auto Quotes" I provide my electronic signature
          and express written consent to telemarketing calls, text messages,
          emails, and postal mail from this Web site,
        </span>
        <NuxtLink class="text-blue-300 underline" to="/partners"
          >our marketing and re-marketing network, and up to eight insurance
          companies or their affiliates</NuxtLink
        ><span>
          or representatives at the phone number (including wireless number),
          email address, and postal address provided by me. I consent to calls
          and text messages transmitting insurance quotes, or seeking related
          additional information from me, using an Automatic Telephone Dialing
          System or prerecorded or artificial voices. I consent that my
          signature is not a condition of purchasing any property, goods, or
          services and that I may revoke my consent at any time.</span
        >
      </div>
      <div>
        <span
          >Additionally, by clicking "Get My Auto Quotes," I acknowledge that I
          have read, understand, and agree to this Web site’s
        </span>
        <NuxtLink class="text-blue-300 underline" to="/privacy-policy"
          >Privacy Policy</NuxtLink
        ><span> and </span>
        <NuxtLink class="text-blue-300 underline" to="/terms-of-use"
          >Terms of Use.</NuxtLink
        >
      </div>
      <div>
        I am directing this Web site to share my information to the providers in
        our network for the purpose of providing me with information about their
        financial services and products.
      </div>
    </div>
  </ValidationObserver>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'nuxt-property-decorator'

import Title from '~/components/Title.vue'
import TextInput from '~/components/TextInput.vue'
import PrimaryButton from '~/components/PrimaryButton.vue'

@Component({
  components: {
    Title,
    TextInput,
    PrimaryButton,
  },
})
export default class GetQuotes extends Vue {
  $refs!: {
    observer: Vue & {
      validateWithInfo: () => Promise<{ isValid: boolean; errors: string[] }>
    }
  }

  errors = {
    address: '',
    phoneNumber: '',
  }

  streetAddress = ''
  phoneNumber = ''

  city = ''
  state = ''

  submitted = false

  // Format phone number
  @Watch('phoneNumber')
  onPhoneNumberChanged(value: any, oldValue: string) {
    // When delete phone number, not work
    if (value.length < oldValue.length) {
      return
    }

    // Just inputed character
    let lastChar = value.slice(-1)

    // Validate is number
    if (isNaN(lastChar)) {
      this.phoneNumber = oldValue
      return
    }

    // Add parentheses
    if (value.length < 2) {
      value = `(${value}`
    }

    if (value.length === 5) {
      value = `${value.slice(0, -1)}) ${lastChar}`
    }

    // Add minus symbol
    if (value.length === 10) {
      value = `${value.slice(0, -1)}-${lastChar}`
    }

    // Limit max length
    if (value.length > 14) {
      value = oldValue
    }

    this.phoneNumber = value
  }
  async created() {
    this.streetAddress = this.$store.state.streetAddress
    this.phoneNumber = this.$store.state.phoneNumber

    const cityState: [string, string] = await this.$store.dispatch(
      'getCityState'
    )
    this.city = cityState[0]
    this.state = cityState[1]
  }

  async submit() {
    this.errors = {
      address: '',
      phoneNumber: '',
    }
    const res = await this.$refs.observer.validateWithInfo()

    if (this.phoneNumber.length < 14) {
      this.errors.phoneNumber = 'Please enter a valid phone number'
    }

    if (!res.isValid) {
      let errors = Object.values(res.errors).flat()
      errors.forEach((error) => {
        if (error.includes('street address') && this.errors.address === '') {
          this.errors.address = error
        }

        if (error.includes('phone number') && this.errors.phoneNumber === '') {
          this.errors.phoneNumber = error
        }
      })

      return
    }

    if (this.errors.phoneNumber !== '') return

    this.submitted = true
    this.$store.commit('SET_STREET_ADDRESS', this.streetAddress)
    this.$store.commit('SET_PHONE_NUMBER', this.phoneNumber)
    this.$emit('submit')
  }
}
</script>
