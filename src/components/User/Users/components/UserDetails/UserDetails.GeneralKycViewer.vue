<template>
  <div class="user-details-general-kyc-viewer">
    <kyc-general-section
      :kyc="kyc"
      :user="user"
    />
  </div>
</template>

<script>
import KycGeneralSection from './UserDetails.Kyc'
import { UserRecord } from '@/js/records/user.record'

export default {
  components: {
    KycGeneralSection,
  },
  props: {
    kyc: {
      type: Object,
      required: true,
    },
    user: {
      type: UserRecord,
      required: true,
    },
  },
}
</script>

<style lang="scss" scoped>
</style>
