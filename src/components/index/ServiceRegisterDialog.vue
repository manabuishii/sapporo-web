<template>
  <v-dialog
    :value="dialogShow"
    overlay-opacity="0.8"
    width="600"
    @click:outside="$emit('close')"
  >
    <v-card>
      <div class="card-header pl-6 pt-4" v-text="'Register Service'" />
      <div class="px-12 pt-2">
        <v-text-field
          v-model="name"
          :error-messages="nameError"
          clearable
          label="Name"
        />
        <v-text-field
          v-model="endpoint"
          :error-messages="endpointError"
          clearable
          label="Endpoint"
          @change="connection = false"
        />
      </div>
      <div class="d-flex justify-end px-12 pb-6 pt-4">
        <v-btn
          :color="checkButtonColor"
          :disabled="connection"
          @click.stop="checkConnection"
          class="mr-4"
          outlined
        >
          <v-icon class="mr-4" v-text="'mdi-access-point-check'" />
          <span v-text="checkButtonText" />
        </v-btn>
        <v-btn
          :disabled="!registerValid || !registerButton"
          color="primary"
          outlined
          @click.stop="submitService"
        >
          <v-icon class="mr-2" v-text="'mdi-arrow-up'" />
          <span v-text="'Submit'" />
        </v-btn>
      </div>
    </v-card>
  </v-dialog>
</template>

<script lang="ts">
import { Service } from '@/store/services'
import { ThisTypedComponentOptionsWithRecordProps } from 'vue/types/options'
import { validUrl } from '@/utils'
import Vue from 'vue'
import { getServiceInfo } from '@/utils/WESRequest'

type Data = {
  name: string
  endpoint: string
  connection: boolean
  registerButton: boolean
  checkButtonColor: string
  checkButtonText: string
}

type Methods = {
  submitService: () => Promise<void>
  checkConnection: () => Promise<void>
}

type Computed = {
  registerValid: boolean
  serviceNames: string[]
  nameError: string
  endpointError: string
}

type Props = {
  dialogShow: boolean
}

const options: ThisTypedComponentOptionsWithRecordProps<
  Vue,
  Data,
  Methods,
  Computed,
  Props
> = {
  props: {
    dialogShow: {
      type: Boolean,
      default: false,
      required: true,
    },
  },

  data() {
    return {
      name: '',
      endpoint: '',
      connection: false,
      registerButton: true,
      checkButtonColor: 'primary',
      checkButtonText: 'Connection Check',
    }
  },

  computed: {
    registerValid() {
      return !this.nameError && !this.endpointError && this.connection
    },

    serviceNames() {
      return this.$store.getters['services/services'].map(
        (service: Service) => service.name
      )
    },

    nameError() {
      if (!this.name) {
        return 'Name is required.'
      }
      if (this.serviceNames.includes(this.name)) {
        return `Service name: ${this.name} already exists.`
      }
      return ''
    },

    endpointError() {
      if (!this.endpoint) {
        return 'Endpoint is required.'
      }
      if (!validUrl(this.endpoint)) {
        return `Endpoint: ${this.endpoint} does not valid.`
      }
      if (!this.connection) {
        return 'Connection check is not completed. Please click Connection Check Button.'
      }
      return ''
    },
  },

  methods: {
    async submitService(): Promise<void> {
      if (this.registerValid) {
        this.registerButton = false
        await this.$store
          .dispatch('services/submitService', {
            name: this.name,
            endpoint: this.endpoint,
            preRegistered: false,
          })
          .then((serviceId) => {
            this.$router.push({ path: '/services', query: { serviceId } })
          })
      }
    },

    async checkConnection(): Promise<void> {
      await getServiceInfo(this.$axios, this.endpoint)
        .then((_) => {
          this.connection = true
        })
        .catch((_) => {
          this.connection = false
          this.checkButtonColor = 'error'
          this.checkButtonText = 'Error'
          setTimeout(() => {
            this.checkButtonColor = 'primary'
          }, 1500)
          setTimeout(() => {
            this.checkButtonText = 'Check Connection'
          }, 1500)
        })
    },
  },
}

export default Vue.extend(options)
</script>
