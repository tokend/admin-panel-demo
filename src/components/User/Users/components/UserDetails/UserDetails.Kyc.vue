<template>
  <div class="user-details-account">
    <h3 class="user-details-account__title">
      {{ "user-details-kyc.header" | globalize }}
    </h3>
    <!--eslint-disable-->
    <div
      class="user-details-account__info"
      v-html="$options.filters.globalize('user-details-kyc.uploaded-document')"
    />
    <!--eslint-enable-->
    <ul class="user-details-account__key-value-list key-value-list">
      <li>
        <span>{{ "user-details-kyc.first-name" | globalize }}</span>
        <span :title="kyc.firstName">
          {{ kyc.firstName }}
        </span>
      </li>
      <li>
        <span>{{ "user-details-kyc.last-name" | globalize }}</span>
        <span :title="kyc.lastName">
          {{ kyc.lastName }}
        </span>
      </li>
      <li>
        <span>{{ "user-details-kyc.date-birth" | globalize }}</span>
        <span :title="kyc.dateOfBirth">
          {{ kyc.dateOfBirth | formatDateDMYT }}
        </span>
      </li>
      <li>
        <span>{{ "user-details-kyc.address-line1" | globalize }}</span>
        <span :title="kyc.address.line1">
          {{ kyc.address.line1 }}
        </span>
      </li>
      <li>
        <span>{{ "user-details-kyc.address-line2" | globalize }}</span>
        <span :title="kyc.address.line2">
          <template v-if="kyc.address.line2">
            {{ kyc.address.line2 }}
          </template>
          <template v-else>
            -
          </template>
        </span>
      </li>
      <li>
        <span>{{ "user-details-kyc.postal-code" | globalize }}</span>
        <span :title="kyc.address.postalCode">
          {{ kyc.address.postalCode }}
        </span>
      </li>
      <li>
        <span>{{ "user-details-kyc.city" | globalize }}</span>
        <span :title="kyc.address.city">
          {{ kyc.address.city }}
        </span>
      </li>
      <li>
        <span>{{ "user-details-kyc.country" | globalize }}</span>
        <span :title="country">
          {{ country }}
        </span>
      </li>
      <li>
        <span>{{ "user-details-kyc.state" | globalize }}</span>
        <span :title="kyc.address.state">
          {{ kyc.address.state }}
        </span>
      </li>
      <li>
        <span>{{ "user-details-kyc.id-document-type" | globalize }}</span>
        <span
          :title="kyc.idDocumentType | idDocumentsVerboseFilter"
        >
          {{ kyc.idDocumentType | idDocumentsVerboseFilter }}
        </span>
      </li>
      <li>
        <span v-if="kyc.documents.kycIdDocument.back">
          {{ "user-details-kyc.id-document-front-side" | globalize }}
        </span>
        <span v-else>
          {{ "user-details-kyc.id-document" | globalize }}
        </span>
        <span>
          <user-doc-link-getter
            :file-key="kyc.documents.kycIdDocument.face"
          >
            {{ "user-details-kyc.open-file" | globalize }}
          </user-doc-link-getter>
        </span>
      </li>
      <li v-if="kyc.documents.kycIdDocument.back">
        <span>{{ "user-details-kyc.id-document-back-side" | globalize }} </span>
        <span>
          <user-doc-link-getter
            :file-key="kyc.documents.kycIdDocument.back"
          >
            {{ "user-details-kyc.open-file" | globalize }}
          </user-doc-link-getter>
        </span>
      </li>
      <li v-if="kyc.documents.kycAvatar">
        <span>{{ "user-details-kyc.avatar" | globalize }}</span>
        <span>
          <user-doc-link-getter :file-key="kyc.documents.kycAvatar">
            {{ "user-details-kyc.open-file" | globalize }}
          </user-doc-link-getter>
        </span>
      </li>
      <li v-if="kyc.documents.kycSelfie">
        <span>
          {{ "user-details-kyc.photo-with-verification-code" | globalize }}
        </span>
        <span>
          <user-doc-link-getter
            :file-key="kyc.documents.kycSelfie"
          >
            {{ "user-details-kyc.open-file" | globalize }}
          </user-doc-link-getter>
        </span>
      </li>
      <li
        class="code-details"
        v-if="kyc.documents.kycSelfie"
      >
        {{ "user-details-kyc.code" | globalize({
          userAddress: user.address.slice(1, 6)
        })
        }}
      </li>
    </ul>
  </div>
</template>

<script>
import { UserDocLinkGetter } from '@comcom/getters'
import { byAlpha2 } from 'iso-country-codes'
import { UserRecord } from '@/js/records/user.record'
export default {
  components: {
    UserDocLinkGetter,
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

  computed: {
    country () {
      const country = Object
        .keys(byAlpha2)
        .find(country => country === this.kyc.address.country)
      return country ? byAlpha2[country].name : ''
    },
  },
}
</script>

<style lang="scss" scoped>
.code-details {
  font-size: 1.2rem;
  margin: 0 0 1rem 0;
  padding-left: 2rem;
}

.user-details-account__title {
  margin-bottom: 2rem;
}

.user-details-account__docs {
  align-items: flex-end;
  display: flex;
  flex-direction: column;
  margin-bottom: 4rem;
}

.user-details-account__doc-view-wrp {
  width: 100%;

  &:not(:first-child) {
    margin-top: 3rem;
  }
}

.user-details-account__info {
  margin-bottom: 3rem;
}

.user-details-account__key-value-list-id-document {
  display: flex;
  flex-direction: column;
}

</style>
